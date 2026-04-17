# AAE5303 Robust Control Technology — Post-Lesson Reflection Report

**Student Name:** [Your Name]  
**Student ID:** [Your Student ID]  
**Group Number:** [Your Group Number]  
**Date:** [Submission Date]  

*Please replace the bracketed fields above before you submit.*

---

## Section 1: AI Usage Experience

During AAE5303, I used **Cursor** as my main coding helper for our group’s **UAV semantic segmentation** track (UAVScenes AMtown02, interval=5). The work was not “one script and done.” It included dataset checks, label remapping, training code, evaluation, and preparing files for the leaderboard.

AI helped most with **boilerplate and structure**: setting up a PyTorch `Dataset`, building a training loop with validation metrics, and wiring **segmentation_models_pytorch** (UNet++ with a ResNet backbone). It also helped when I was stuck on **environment issues** (CUDA / driver warnings, missing packages) and when I needed to **read error traces** quickly.

I used Cursor **often during heavy coding weeks**, and **less often** when I was only running long training jobs. The features I used most were the **chat** (for explaining a file or an error) and **edits across several files** (for example `train.py`, `dataset.py`, and `evaluate_submission.py` together). Inline completion was useful for small fixes, like tensor shapes or argument names.

---

## Section 2: Understanding AI Limitations

One clear limitation showed up when I tried to **publish our code to GitHub**. The assistant set the remote to my account’s repo, but the machine’s SSH key was tied to **another GitHub user**. The push failed with a “permission denied” message. The AI could describe *what* SSH keys are, but it could not **see my GitHub account settings** or **know which key was active** on my PC. The wrong fix would have been to keep retrying the same push.

I caught the problem from the **exact error text** in the terminal (which user GitHub thought I was). The fix was practical: **add the correct SSH key to my own account** and point `github.com` to the right `IdentityFile` in `~/.ssh/config`. This was a good reminder: AI can suggest commands, but **identity and permissions** are my responsibility.

A second limitation was **metric confusion**. Early on, I thought the course leaderboard used a simple weighted mix of raw Dice, mIoU, and FWIoU. The AI sometimes reasoned in that same simple way. Later I learned the instructor used **normalized scores** before weighting. AI did not “know” the hidden rule unless I pasted it. I adjusted my goals once I understood that.

---

## Section 3: Engineering Validation

For segmentation, “looks fine” is not enough. I tried to validate in a few concrete ways:

1. **Data pairing:** I checked that each RGB image had a matching mask name, and that resize used **nearest** for masks (so labels did not get mixed by interpolation).  
2. **Training vs validation:** I watched **loss and mIoU on a held-out split** (and later K-fold setups) so I did not only trust training loss.  
3. **Metrics from confusion matrix:** I relied on the same definitions the course cares about (Dice, mIoU, FWIoU) instead of only pixel accuracy.  
4. **Visual checks:** I exported **input / ground truth / prediction** images for one frame. Wrong colors or shifted masks are easy to spot by eye.

These steps caught real issues, such as shape errors for models that need height and width divisible by 16 or 32, and bad habits like trusting a single number without plots.

---

## Section 4: Problem-Solving Process

A major challenge was pushing **reliable scores** under **class imbalance** (large background and small classes like “pool”). A small model trained fast but plateaued. Moving to **UNet++ with a pretrained encoder** helped. Combining **weighted cross-entropy** with a **Dice-style term** also helped the model care more about overlap, not only per-pixel labels.

Another challenge was **time and hardware**. Training at full resolution was slow. We used a smaller **scale** (resize factor) for experiments, then kept training and evaluation **consistent** when comparing runs.

AI helped by drafting code and suggesting libraries. It sometimes failed when the problem needed **my own judgment**—for example, whether **test-time augmentation** helped or hurt on our validation setup. I had to run experiments and compare numbers myself.

---

## Section 5: Learning Growth

At the start, I treated segmentation as “call a model and train.” Now I pay more attention to **labels, ignores, class weights, and metrics**. I am more comfortable with a **full mini-pipeline**: data → train → evaluate → export JSON.

I also improved at **reading errors** (CUDA, shape mismatch, SSH) instead of panicking. The course pushed me to connect **perception** (what the UAV sees) with **downstream use** (maps or planning), even if my own module was only segmentation.

---

## Section 6: Critical Reflection

AI **sped me up** on repetitive coding and explained unfamiliar APIs. It could **slow learning** if I pasted suggestions without reading them. The segmentation task was large enough that I still had to **think**: split strategy, loss weights, when to ensemble, and what to submit.

Next time I would set two rules for myself:  
(1) **Always verify** data and metrics on a small batch before a long run.  
(2) For anything that touches **grades or submission format**, read the **official instructions** first, then use AI as a helper—not the other way around.

---

## Section 7: Evidence (Optional)

### 7.1 Short example (SSH lesson)

Wrong mental model: “If `git push` fails, keep changing commands at random.”  
Better model: “Read **who** GitHub thinks I am, then fix **keys and config**.”

### 7.2 Prompt pattern that worked

**Me:** “Explain this traceback line by line and list the three most likely causes.”  
**Why it worked:** It forced a structured answer I could check against the real log.

### 7.3 Prompt pattern that needed care

**Me:** “How is the leaderboard score calculated?”  
**Risk:** If I do not paste the instructor’s formula, the answer may be **generic** and **wrong for our course**. I learned to **paste the rule** or say “I do not know the exact formula.”

---

**Word count note:** The body above (Sections 1–6) is written to stay close to the **800–1200 word** course requirement. If your checker counts differently, trim Section 7 first or shorten Section 4 slightly.
