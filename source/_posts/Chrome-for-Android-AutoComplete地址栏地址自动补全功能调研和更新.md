---
title: Chrome for Android AutoComplete地址栏地址自动补全功能调研和更新
date: 2018-03-09 14:39:58
tags: 
- Android
- Chrome
- Gboard
categories: 
- 移动开发
---
# Chrome地址栏地址自动补全功能调研和更新
[CSDN文章对应地址](http://blog.csdn.net/lwq573384928/article/details/79484817)
## Chrome地址栏地址自动补全功能预览
![Chrome](http://7xruee.com1.z0.glb.clouddn.com/video2gif_20180308_152921.gif)
补全前提：
1. 使用Gboard输入法
2. Gboard输入法打开了【文字更正】功能里面的现实建议栏
补全逻辑：
1. 输入一个字符后，将推荐的第一个链接补全，补全的文字为蓝色背景选中状态，并且用户自己输入的文本下面有下划线（这个下划线不是自己通过SpannableString来实现的，应该是EditText自己为待用户选择推荐（commitText）的的文本默认添加的）
2. 输入一个非字母类型的，会自动commitText，但是此时把非字母类型的字符删除后，输入法的输入建议会再次出现，并且之前的用户输入文本会再次出现下划线，这种状态其实就是带选择状态，可以通过输入法最顶部的推荐直接替换文字。
3. 选择了输入法的输入建议文本，浏览器也能继续进行输入推荐和自动补全

## 最初版本情况
Chrome之前的版本其实不是这样的，而是类似于禁用了输入法的输入建议，而直接替换输入框的文本，再将一部分文字设置为select状态。
代码实现参考，[Android EditText 通过TextWatcher实现自动补全的注意点](http://blog.csdn.net/lwq573384928/article/details/79317064)

## Chrome最新版代码

### 实现效果
![自己实现的效果图](http://7xruee.com1.z0.glb.clouddn.com/video2gif_20180308_151910.gif)

### 实现代码
主要核心代码，忽略了一些变量的定义，详细参考Chrome源码即可
```
public void onAutoComplete(String suggestion) {
        String userText = getTextWithoutAutocomplete();
        suggestion = suggestion.startsWith(userText) ? suggestion.replaceFirst(userText, "") : "";
        if (shouldAutocomplete()) {
            setAutocompleteText(userText, suggestion);
        }
    }

    private void limitDisplayableLength() {
        // To limit displayable length we replace middle portion of the string with ellipsis.
        // That affects only presentation of the text, and doesn't affect other aspects like
        // copying to the clipboard, getting text with getText(), etc.
        final int maxLength = MAX_DISPLAYABLE_LENGTH_END;

        Editable text = getText();
        int textLength = text.length();
        if (textLength <= maxLength) {
            if (mDidEllipsizeTextHint) {
                EllipsisSpan[] spans = text.getSpans(0, textLength, EllipsisSpan.class);
                if (spans != null && spans.length > 0) {
                    assert spans.length == 1 : "Should never apply more than a single EllipsisSpan";
                    for (int i = 0; i < spans.length; i++) {
                        text.removeSpan(spans[i]);
                    }
                }
            }
            mDidEllipsizeTextHint = false;
            return;
        }

        mDidEllipsizeTextHint = true;

        int spanLeft = text.nextSpanTransition(0, textLength, EllipsisSpan.class);
        if (spanLeft != textLength) return;

        spanLeft = maxLength / 2;
        text.setSpan(EllipsisSpan.INSTANCE, spanLeft, textLength - spanLeft,
                Editable.SPAN_INCLUSIVE_EXCLUSIVE);
    }

    public boolean shouldAutocomplete() {
        boolean retVal = mBatchEditNestCount == 0 && mLastEditWasTyping
                && mCurrentState.isCursorAtEndOfUserText() && !isKeyboardBlacklisted()
                && isNonCompositionalText(getTextWithoutAutocomplete());
        Log.i(TAG, "shouldAutocomplete: " + retVal);
        return retVal;
    }

    private boolean isKeyboardBlacklisted() {
        String pkgName = getKeyboardPackageName();
        return pkgName.contains(".iqqi") // crbug.com/767016
                || pkgName.contains("omronsoft") || pkgName.contains(".iwnn"); // crbug.com/758443
    }

    public String getKeyboardPackageName() {
        String defaultIme = Settings.Secure.getString(
                getContext().getContentResolver(), Settings.Secure.DEFAULT_INPUT_METHOD);
        return defaultIme == null ? "" : defaultIme;
    }

    public static boolean isNonCompositionalText(String text) {
        // To start with, we are only activating this for English alphabets, European characters,
        // numbers and URLs to avoid potential bad interactions with more complex IMEs.
        // The rationale for including character sets with diacritical marks is that backspacing on
        // a letter with a diacritical mark most likely deletes the whole character instead of
        // removing the diacritical mark.
        // TODO(changwan): also scan for other traditionally non-IME charsets.
        return NON_COMPOSITIONAL_TEXT_PATTERN.matcher(text).matches();
    }

    @Override
    protected void onSelectionChanged(int selStart, int selEnd) {
        if (mCurrentState == null) {
            return;
        }
        if (mCurrentState.getSelStart() == selStart && mCurrentState.getSelEnd() == selEnd) return;

        mCurrentState.setSelection(selStart, selEnd);
        if (mBatchEditNestCount > 0) return;
        int len = mCurrentState.getUserText().length();
        if (mCurrentState.hasAutocompleteText()) {
            if (selStart > len || selEnd > len) {
                Log.i(TAG, "Autocomplete text is being touched. Make it real.");
                if (mInputConnection != null) mInputConnection.commitAutocomplete();
            } else {
                Log.i(TAG, "Touching before the cursor removes autocomplete.");
                clearAutocompleteTextAndUpdateSpanCursor();
            }
        }
        notifyAutocompleteTextStateChanged();
    }

    private void clearAutocompleteTextAndUpdateSpanCursor() {
        Log.i(TAG, "clearAutocompleteAndUpdateSpanCursor");
        clearAutocompleteText();
        // Take effect and notify if not already in a batch edit.
        if (mInputConnection != null) {
            mInputConnection.onBeginImeCommand();
            mInputConnection.onEndImeCommand();
        } else {
            mSpanCursorController.removeSpan();
            notifyAutocompleteTextStateChanged();
        }
    }
class AutocompleteState {
        private String mUserText;
        private String mAutocompleteText;
        private int mSelStart;
        private int mSelEnd;

        public AutocompleteState(AutocompleteState a) {
            copyFrom(a);
        }

        public AutocompleteState(String userText, String autocompleteText, int selStart, int selEnd) {
            set(userText, autocompleteText, selStart, selEnd);
        }

        public void set(String userText, String autocompleteText, int selStart, int selEnd) {
            mUserText = userText;
            mAutocompleteText = autocompleteText;
            mSelStart = selStart;
            mSelEnd = selEnd;
        }

        public void copyFrom(AutocompleteState a) {
            set(a.mUserText, a.mAutocompleteText, a.mSelStart, a.mSelEnd);
        }

        public String getUserText() {
            return mUserText;
        }

        public String getAutocompleteText() {
            return mAutocompleteText;
        }

        public boolean hasAutocompleteText() {
            return !TextUtils.isEmpty(mAutocompleteText);
        }

        /** @return The whole text including autocomplete text. */
        public String getText() {
            return mUserText + mAutocompleteText;
        }

        public int getSelStart() {
            return mSelStart;
        }

        public int getSelEnd() {
            return mSelEnd;
        }

        public void setSelection(int selStart, int selEnd) {
            mSelStart = selStart;
            mSelEnd = selEnd;
        }

        public void setUserText(String userText) {
            mUserText = userText;
        }

        public void setAutocompleteText(String autocompleteText) {
            mAutocompleteText = autocompleteText;
        }

        public void clearAutocompleteText() {
            mAutocompleteText = "";
        }

        public boolean isCursorAtEndOfUserText() {
            return mSelStart == mUserText.length() && mSelEnd == mUserText.length();
        }

        public boolean isWholeUserTextSelected() {
            return mSelStart == 0 && mSelEnd == mUserText.length();
        }

        /**
         * @param prevState The previous state to compare the current state with.
         * @return Whether the current state is backward-deleted from prevState.
         */
        public boolean isBackwardDeletedFrom(AutocompleteState prevState) {
            return isCursorAtEndOfUserText() && prevState.isCursorAtEndOfUserText()
                    && isPrefix(mUserText, prevState.mUserText);
        }

        /**
         * @param prevState The previous state to compare the current state with.
         * @return Whether the current state is forward-typed from prevState.
         */
        public boolean isForwardTypedFrom(AutocompleteState prevState) {
            return isCursorAtEndOfUserText() && prevState.isCursorAtEndOfUserText()
                    && isPrefix(prevState.mUserText, mUserText);
        }

        /**
         * @param prevState The previous state to compare the current state with.
         * @return The differential string that has been backward deleted.
         */
        public String getBackwardDeletedTextFrom(AutocompleteState prevState) {
            if (!isBackwardDeletedFrom(prevState)) return null;
            return prevState.mUserText.substring(mUserText.length());
        }

        public boolean isPrefix(String a, String b) {
            return b.startsWith(a) && b.length() > a.length();
        }

        /**
         * When the user manually types the next character that was already suggested in the previous
         * autocomplete, then the suggestion is still valid if we simply remove one character from the
         * beginning of it. For example, if prev = "a[bc]" and current text is "ab", this method
         * constructs "ab[c]".
         * @param prevState The previous state.
         * @return Whether the shifting was successful.
         */
        public boolean reuseAutocompleteTextIfPrefixExtension(AutocompleteState prevState) {
            // Shift when user text has grown or remains the same, but still prefix of prevState's whole
            // text.
            int diff = mUserText.length() - prevState.mUserText.length();
            if (diff < 0) return false;
            if (!isPrefix(mUserText, prevState.getText())) return false;
            mAutocompleteText = prevState.mAutocompleteText.substring(diff);
            return true;
        }

        public void commitAutocompleteText() {
            mUserText += mAutocompleteText;
            mAutocompleteText = "";
        }

        @Override
        public boolean equals(Object o) {
            if (!(o instanceof AutocompleteState)) return false;
            if (o == this) return true;
            AutocompleteState a = (AutocompleteState) o;
            return mUserText.equals(a.mUserText) && mAutocompleteText.equals(a.mAutocompleteText)
                    && mSelStart == a.mSelStart && mSelEnd == a.mSelEnd;
        }

        @Override
        public int hashCode() {
            return mUserText.hashCode() * 2 + mAutocompleteText.hashCode() * 3 + mSelStart * 5
                    + mSelEnd * 7;
        }

        @Override
        public String toString() {
            return String.format(Locale.US, "AutocompleteState {[%s][%s] [%d-%d]}", mUserText,
                    mAutocompleteText, mSelStart, mSelEnd);
        }
    }
    public boolean hasAutocomplete() {
        boolean retVal = mCurrentState.hasAutocompleteText();
        Log.i(TAG, "hasAutocomplete: " + retVal);
        return retVal;
    }


    private class AutocompleteInputConnection extends InputConnectionWrapper {
        private final AutocompleteState mPreBatchEditState;

        public AutocompleteInputConnection() {
            super(null, true);
            mPreBatchEditState = new AutocompleteState(mCurrentState);
        }

        private boolean incrementBatchEditCount() {
            ++mBatchEditNestCount;
            // After the outermost super.beginBatchEdit(), EditText will stop selection change
            // update to the IME app.
            return super.beginBatchEdit();
        }

        private boolean decrementBatchEditCount() {
            --mBatchEditNestCount;
            return super.endBatchEdit();
        }

        public void commitAutocomplete() {
            Log.i(TAG, "commitAutocomplete");
            if (!hasAutocomplete()) return;
            mCurrentState.commitAutocompleteText();
            // Invalidate mPreviouslySetState.
            mPreviouslySetState.copyFrom(mCurrentState);
            mLastEditWasTyping = false;
            incrementBatchEditCount(); // avoids additional notifyAutocompleteTextStateChanged()
            mSpanCursorController.commitSpan();
            decrementBatchEditCount();
        }

        @Override
        public boolean beginBatchEdit() {
            Log.i(TAG, "beginBatchEdit");
            onBeginImeCommand();
            boolean retVal = incrementBatchEditCount();
            onEndImeCommand();
            return retVal;
        }

        /**
         * Always call this at the beginning of any IME command. Compare this with beginBatchEdit()
         * which is by itself an IME command.
         * @return Whether the call was successful.
         */
        public boolean onBeginImeCommand() {
            Log.i(TAG, "onBeginImeCommand: " + mBatchEditNestCount);
            boolean retVal = incrementBatchEditCount();
            if (mBatchEditNestCount == 1) {
                mPreBatchEditState.copyFrom(mCurrentState);
            } else if (mDeletePostfixOnNextBeginImeCommand > 0) {
                int len = getText().length();
                getText().delete(len - mDeletePostfixOnNextBeginImeCommand, len);
            }
            mDeletePostfixOnNextBeginImeCommand = 0;
            mSpanCursorController.removeSpan();
            return retVal;
        }

        private void restoreBackspacedText(String diff) {
            Log.i(TAG, "restoreBackspacedText. diff: " + diff);

            if (mBatchEditNestCount > 0) {
                // If batch edit hasn't finished, we will restore backspaced text only for visual
                // effects. However, for internal operations to work correctly, we need to remove
                // the restored diff at the beginning of next IME operation.
                mDeletePostfixOnNextBeginImeCommand = diff.length();
            }
            if (mBatchEditNestCount == 0) { // only at the outermost batch edit
                if (shouldFinishCompositionOnDeletion()) super.finishComposingText();
            }
            incrementBatchEditCount(); // avoids additional notifyAutocompleteTextStateChanged()
            Editable editable = getEditableText();
            editable.append(diff);
            decrementBatchEditCount();
        }

        private boolean setAutocompleteSpan() {
            mSpanCursorController.removeSpan();
            if (!mCurrentState.isCursorAtEndOfUserText()) return false;

            if (mCurrentState.reuseAutocompleteTextIfPrefixExtension(mPreviouslySetState)) {
                mSpanCursorController.setSpan(mCurrentState);
                return true;
            } else {
                return false;
            }
        }

        @Override
        public boolean endBatchEdit() {
            Log.i(TAG, "endBatchEdit");
            onBeginImeCommand();
            boolean retVal = decrementBatchEditCount();
            onEndImeCommand();
            return retVal;
        }

        /**
         * Always call this at the end of an IME command. Compare this with endBatchEdit()
         * which is by itself an IME command.
         * @return Whether the call was successful.
         */
        public boolean onEndImeCommand() {
            Log.i(TAG, "onEndImeCommand: " + (mBatchEditNestCount - 1));
            String diff = mCurrentState.getBackwardDeletedTextFrom(mPreBatchEditState);
            if (diff != null) {
                // Update selection first such that keyboard app gets what it expects.
                boolean retVal = decrementBatchEditCount();

                if (mPreBatchEditState.hasAutocompleteText()) {
                    // Undo delete to retain the last character and only remove autocomplete text.
                    restoreBackspacedText(diff);
                }
                mLastEditWasTyping = false;
                clearAutocompleteText();
                notifyAutocompleteTextStateChanged();
                return retVal;
            }
            if (!setAutocompleteSpan()) {
                clearAutocompleteText();
            }
            boolean retVal = decrementBatchEditCount();
            // Simply typed some characters or whole text selection has been overridden.
            if (mCurrentState.isForwardTypedFrom(mPreBatchEditState)
                    || (mPreBatchEditState.isWholeUserTextSelected()
                    && mCurrentState.getUserText().length() > 0
                    && mCurrentState.isCursorAtEndOfUserText())) {
                mLastEditWasTyping = true;
            }
            notifyAutocompleteTextStateChanged();
            return retVal;
        }

        @Override
        public boolean commitText(CharSequence text, int newCursorPosition) {
            Log.i(TAG, "commitText: " + text);
            onBeginImeCommand();
            boolean retVal = super.commitText(text, newCursorPosition);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean setComposingText(CharSequence text, int newCursorPosition) {
            Log.i(TAG, "setComposingText: " + text);
            onBeginImeCommand();
            boolean retVal = super.setComposingText(text, newCursorPosition);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean setComposingRegion(int start, int end) {
            onBeginImeCommand();
            boolean retVal = super.setComposingRegion(start, end);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean finishComposingText() {
            Log.i(TAG, "finishComposingText");
            onBeginImeCommand();
            boolean retVal = super.finishComposingText();
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean deleteSurroundingText(final int beforeLength, final int afterLength) {
            onBeginImeCommand();
            boolean retVal = super.deleteSurroundingText(beforeLength, afterLength);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean setSelection(final int start, final int end) {
            onBeginImeCommand();
            boolean retVal = super.setSelection(start, end);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean performEditorAction(final int editorAction) {
            Log.i(TAG, "performEditorAction: " + editorAction);
            onBeginImeCommand();
            commitAutocomplete();
            boolean retVal = super.performEditorAction(editorAction);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean sendKeyEvent(final KeyEvent event) {
            Log.i(TAG, "sendKeyEvent: " + event.getKeyCode());
            onBeginImeCommand();
            boolean retVal = super.sendKeyEvent(event);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public ExtractedText getExtractedText(final ExtractedTextRequest request, final int flags) {
            Log.i(TAG, "getExtractedText");
            onBeginImeCommand();
            ExtractedText retVal = super.getExtractedText(request, flags);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public CharSequence getTextAfterCursor(final int n, final int flags) {
            Log.i(TAG, "getTextAfterCursor");
            onBeginImeCommand();
            CharSequence retVal = super.getTextAfterCursor(n, flags);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public CharSequence getTextBeforeCursor(final int n, final int flags) {
            Log.i(TAG, "getTextBeforeCursor");
            onBeginImeCommand();
            CharSequence retVal = super.getTextBeforeCursor(n, flags);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public CharSequence getSelectedText(final int flags) {
            Log.i(TAG, "getSelectedText");
            onBeginImeCommand();
            CharSequence retVal = super.getSelectedText(flags);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean commitCompletion(CompletionInfo text) {
            Log.i(TAG, "commitCompletion");
            onBeginImeCommand();
            boolean retVal = super.commitCompletion(text);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean commitContent(InputContentInfo inputContentInfo, int flags, Bundle opts) {
            Log.i(TAG, "commitContent");
            onBeginImeCommand();
            boolean retVal = super.commitContent(inputContentInfo, flags, opts);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean commitCorrection(CorrectionInfo correctionInfo) {
            Log.i(TAG, "commitCorrection");
            onBeginImeCommand();
            boolean retVal = super.commitCorrection(correctionInfo);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean deleteSurroundingTextInCodePoints(int beforeLength, int afterLength) {
            Log.i(TAG, "deleteSurroundingTextInCodePoints");
            onBeginImeCommand();
            boolean retVal = super.deleteSurroundingTextInCodePoints(beforeLength, afterLength);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public int getCursorCapsMode(int reqModes) {
            Log.i(TAG, "getCursorCapsMode");
            onBeginImeCommand();
            int retVal = super.getCursorCapsMode(reqModes);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean requestCursorUpdates(int cursorUpdateMode) {
            Log.i(TAG, "requestCursorUpdates");
            onBeginImeCommand();
            boolean retVal = super.requestCursorUpdates(cursorUpdateMode);
            onEndImeCommand();
            return retVal;
        }

        @Override
        public boolean clearMetaKeyStates(int states) {
            Log.i(TAG, "clearMetaKeyStates");
            onBeginImeCommand();
            boolean retVal = super.clearMetaKeyStates(states);
            onEndImeCommand();
            return retVal;
        }
    }

    private boolean shouldFinishCompositionOnDeletion() {
        String defaultIme = Settings.Secure.getString(
                getContext().getContentResolver(), Settings.Secure.DEFAULT_INPUT_METHOD);
        defaultIme = defaultIme == null ? "" : defaultIme;

        return !defaultIme.contains("com.sec.android.inputmethod");
    }

    private int mDeletePostfixOnNextBeginImeCommand;

    private void clearAutocompleteText() {
        Log.i(TAG, "clearAutocomplete");
        mPreviouslySetState.clearAutocompleteText();
        mCurrentState.clearAutocompleteText();
    }

    private final AutocompleteState mPreviouslyNotifiedState;
    private void notifyAutocompleteTextStateChanged() {

        if (mBatchEditNestCount > 0) {
            // crbug.com/764749
            Log.w(TAG, "Did not notify - in batch edit.");
            return;
        }
        if (mCurrentState.equals(mPreviouslyNotifiedState)) {
            // crbug.com/764749
            Log.w(TAG, "Did not notify - no change.");
            return;
        }
        if (mCurrentState.getUserText().equals(mPreviouslyNotifiedState.getUserText())
                && (mCurrentState.hasAutocompleteText()
                || !mPreviouslyNotifiedState.hasAutocompleteText())) {
            // Nothing has changed except that autocomplete text has been set or modified. Or
            // selection change did not affect autocomplete text. Autocomplete text is set by the
            // controller, so only text change or deletion of autocomplete text should be notified.
            mPreviouslyNotifiedState.copyFrom(mCurrentState);
            return;
        }
        mPreviouslyNotifiedState.copyFrom(mCurrentState);
        if (mIgnoreTextChangeFromAutocomplete) {
            // crbug.com/764749
            Log.w(TAG, "Did not notify - ignored.");
            return;
        }
        // The current model's mechanism always moves the cursor at the end of user text, so we
        // don't need to update the display.
        onAutocompleteTextStateChanged(false /* updateDisplay */);
    }
    private static class SpanCursorController {
        private final PopUpWindowInAutoCompleteTextView mDelegate;
        private BackgroundColorSpan mSpan;

        public SpanCursorController(PopUpWindowInAutoCompleteTextView delegate) {
            mDelegate = delegate;
        }

        public void setSpan(AutocompleteState state) {
            int sel = state.getSelStart();

            if (mSpan == null) mSpan = new BackgroundColorSpan(mDelegate.getHighlightColor());
            SpannableString spanString = new SpannableString(state.getAutocompleteText());
            // The flag here helps make sure that span does not get spill to other part of the text.
            spanString.setSpan(mSpan, 0, state.getAutocompleteText().length(),
                    Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
            Editable editable = mDelegate.getEditableText();
            editable.append(spanString);

            // Keep the original selection before adding spannable string.
            Selection.setSelection(editable, sel, sel);
            setCursorVisible(false);
            Log.i(TAG, "setSpan: " + getEditableDebugString(editable));
        }

        private void setCursorVisible(boolean visible) {
            if (mDelegate.isFocused()) mDelegate.setCursorVisible(visible);
        }

        private int getSpanIndex(Editable editable) {
            if (editable == null || mSpan == null) return -1;
            return editable.getSpanStart(mSpan); // returns -1 if mSpan is not attached
        }

        public void reset() {
            setCursorVisible(true);
            Editable editable = mDelegate.getEditableText();
            int idx = getSpanIndex(editable);
            if (idx != -1) {
                editable.removeSpan(mSpan);
            }
            mSpan = null;
        }

        public boolean removeSpan() {
            setCursorVisible(true);
            Editable editable = mDelegate.getEditableText();
            int idx = getSpanIndex(editable);
            if (idx == -1) return false;
            editable.removeSpan(mSpan);
            editable.delete(idx, editable.length());
            mSpan = null;
            {
                Log.i(TAG, "removeSpan - after removal: " + getEditableDebugString(editable));
            }
            return true;
        }

        public void commitSpan() {
            mDelegate.getEditableText().removeSpan(mSpan);
            setCursorVisible(true);
        }

        public void reflectTextUpdateInState(AutocompleteState state, CharSequence text) {
            if (text instanceof Editable) {
                Editable editable = (Editable) text;
                int idx = getSpanIndex(editable);
                if (idx != -1) {
                    // We do not set autocomplete text here as model should solely control it.
                    state.setUserText(editable.subSequence(0, idx).toString());
                    return;
                }
            }
            state.setUserText(text.toString());
        }
    }
    /**
     * @param editable The editable.
     * @return Debug string for the given {@Editable}.
     */
    private static String getEditableDebugString(Editable editable) {
        return String.format(Locale.US, "Editable {[%s] SEL[%d %d] COM[%d %d]}",
                editable.toString(), Selection.getSelectionStart(editable),
                Selection.getSelectionEnd(editable),
                BaseInputConnection.getComposingSpanStart(editable),
                BaseInputConnection.getComposingSpanEnd(editable));
    }

    @Override
    protected void onTextChanged(CharSequence text, int start, int lengthBefore, int lengthAfter) {
        Log.i(TAG, "onTextChanged: " + text);
        if(mSpanCursorController==null){
            return;
        }
        mSpanCursorController.reflectTextUpdateInState(mCurrentState, text);
        if (mBatchEditNestCount > 0) return; // let endBatchEdit() handles changes from IME.
        // An external change such as text paste occurred.
        mLastEditWasTyping = false;
        clearAutocompleteTextAndUpdateSpanCursor();
    }

    public String getTextWithoutAutocomplete() {
        String retVal = mCurrentState.getUserText();
        Log.i(TAG, "getTextWithoutAutocomplete: " + retVal);
        return retVal;
    }

    public void onAutocompleteTextStateChanged(boolean updateDisplay) {
        if (updateDisplay) limitDisplayableLength();
        onTextChangedForAutocomplete();
    }

    public void onTextChangedForAutocomplete() {
	    //触发搜索建议获取的逻辑
    }

    /**
     * Sets whether text changes should trigger autocomplete.
     * <p>
     * enabling autocomplete.
     *
     * @param ignoreAutocomplete Whether text changes should be ignored and no auto complete
     *                           triggered.
     */
    public void setIgnoreTextChangeFromAutocomplete(boolean ignoreAutocomplete) {

        mIgnoreTextChangeFromAutocomplete = ignoreAutocomplete;
    }

    public void setAutocompleteText(CharSequence userText, CharSequence inlineAutocompleteText) {
        setAutocompleteTextInternal(userText.toString(), inlineAutocompleteText.toString());
    }

    private void setAutocompleteTextInternal(String userText, String autocompleteText) {
        mPreviouslySetState.set(userText, autocompleteText, userText.length(), userText.length());
        // TODO(changwan): avoid any unnecessary removal and addition of autocomplete text when it
        // is not changed or when it is appended to the existing autocomplete text.
        if (mInputConnection != null) {
            mInputConnection.onBeginImeCommand();
            mInputConnection.onEndImeCommand();
        }
    }

    @Override
    public InputConnection onCreateInputConnection(EditorInfo outAttrs) {
	    //创建InputConnection
        mBatchEditNestCount = 0;
        mInputConnection = new AutocompleteInputConnection();
        mInputConnection.setTarget(super.onCreateInputConnection(outAttrs));
        return mInputConnection;
    }
```

-----
Have Fun :)


