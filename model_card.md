# Model Card: Mood Machine

This model card is for the Mood Machine project, which includes **two** versions of a mood classifier:

1. A **rule based model** implemented in `mood_analyzer.py`
2. A **machine learning model** implemented in `ml_experiments.py` using scikit learn

You may complete this model card for whichever version you used, or compare both if you explored them.

## 1. Model Overview

**Model type:**  
 - Only used the rule based model, since training a model   would take longer but would be more effective at handling the edge cases like sarcasm.

**Intended purpose:**  
- This model is trying to classify captions with positive or negative feelings, based on the words and phrasing.

**How it works (brief):**  
- The rule-based model tokenizes input text (lowercase, split on spaces, punctuation stripped) and scans each token against two word lists: POSITIVE_WORDS and NEGATIVE_WORDS. Each positive word match adds +1 to the score; each negative word match subtracts 1.

    - Two additional rules improve accuracy:

        - Negation handling — if a negation word (e.g. "not", "never", "don't") immediately precedes a sentiment word, the contribution is flipped (e.g. "not happy" scores -1 instead of +1).

        - Sarcasm detection — if the text contains at least one positive word alongside a known sarcasm trigger word (e.g. "stuck", "traffic", "terrible"), the score is forced negative, catching phrases like "I love getting stuck in traffic."

        - The final score is mapped to a label: positive (> 0), negative (< 0), or neutral (== 0).


## 2. Data

**Dataset description:**  
- There are 11 posts in Sample_Posts and I added new ones based on the previous posts but tried to make them more unqiue and more accurate to what someone might say currently.

**Labeling process:**  
- I used the words to label, if the words or sentence was positive as a whole, I labeled it accordingly. For hard to label posts I just put mixed but that is cause I am a human able to interpret it would be hard for a rule based system to come to that same conclusion. 

**Important characteristics of your dataset:**  

- Contains informal/conversational language and sports references (e.g. "I am cooked", "Met Mbappe")
Includes mixed-feeling posts where both positive and negative words appear (ex. "Feeling tired but kind of hopeful")
- Contains short, ambiguous posts where tone depends on context (ex. "On to the next one", "This is fine")
- Leaning towards positive: 7 of 12 posts are labeled positive, only 3 negative

**Possible issues with the dataset:**  

- The dataset is very small (12 posts), making accuracy scores unreliable, one wrong prediction swings accuracy.
- Most posts use mainstream English, slang, emojis, and non-English expressions are underrepresented.
- Some labels are debatable, "On to the next one" could reasonably be neutral or mixed depending on context.

## 3. How the Rule Based Model Works (if used)

**Your scoring rules:**  

- Each positive word match adds +1 to the score; each negative word match subtracts 1.
Negation handling: if a negation word ("not", "never", "don't", etc.) immediately precedes a sentiment word, the contribution is flipped, "not happy" scores -1 instead of +1.
- Sarcasm detection: if the text contains both a positive word and a known sarcasm trigger word ("stuck", "traffic", "terrible", etc.), the score is forced negative regardless of the raw total.
- Label thresholds: score > 0 → positive, score < 0 → negative, score == 0 → neutral. No "mixed" label is produced by the rule-based model.

**Strengths of this approach:**  
- Transparent and easy to debug — you can see exactly which words drove the prediction.

- Handles straightforward positive/negative sentences reliably when the key word is in the list.

- Negation handling correctly flips sentiment for common patterns like "not happy" or "don't love."

**Weaknesses of this approach:**  
- Cannot produce a "mixed" label, so posts with both positive and negative words often land on whichever side has more matches rather than reflecting true ambiguity.
- Entirely dependent on the word lists — any unfamiliar slang, emojis, or context-specific language ("I am cooked") scores neutral unless manually added.
- Sarcasm detection only works for pre-defined trigger words; novel sarcastic phrasing will be missed.

## 4. How the ML Model Works (if used)

## 5. Evaluation

**How you evaluated the model:**  
The model was evaluated on all 12 posts in `SAMPLE_POSTS` by comparing `predict_label` output against `TRUE_LABELS`. The rule-based model achieved ~75% accuracy on the sample set.

**Examples of correct predictions:**

- “I love this class so much” → positive. “love” is in POSITIVE_WORDS and no negation or sarcasm trigger is present.
- “Today was a terrible day” → negative. “terrible” is in NEGATIVE_WORDS and scores -1.
- “I am not happy about this” → negative. Negation handling flips “happy” from +1 to -1.

**Examples of incorrect predictions:**

- “Feeling tired but kind of hopeful” → predicted neutral, true mixed. “tired” (-1) and “hopeful” (+1) cancel out to 0, and the model has no “mixed” label.
- “On to the next one” → predicted neutral, true positive. None of the words appear in either word list, so the score is 0.

## 6. Limitations

- The dataset is very small (12 posts), so accuracy numbers are not statistically meaningful.
- The model cannot produce a “mixed” label, missing posts that express both positive and negative feelings.
- Coverage depends entirely on the manually curated word lists — any word not in the list contributes nothing to the score.
- Sarcasm detection only works for a fixed set of trigger words and will miss novel sarcastic phrasing.

## 7. Ethical Considerations

Misclassifying a message expressing distress as neutral or positive could be harmful in any application that routes users to support resources. The word lists reflect a particular style of English, so posts using slang from other language communities or non-English expressions may be systematically misread. 
## 8. Ideas for Improvement

- Expand the dataset with more diverse examples including slang, emojis, and sarcasm to improve coverage.
- Weight high-intensity words more heavily ex. “hate” = -2, “annoyed” = -1
- Add emoji and common slang mappings: “lol” and “💀” as explicit sentiment signals.
- Replace the rule-based approach with a small ML classifier trained on a larger labeled dataset for better generalization.
