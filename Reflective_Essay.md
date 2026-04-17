# AAE5303 Robust Control Technology — Post-Lesson Reflection Report

**Student Name:** Ding Zhongwei  
**Student ID:** 25059382G  
**Group Number:** 12 VisLangFly  
**Date:** 2026/4/17  

---

## Section 1: AI Usage Experience

During AAE5303, I used **Cursor** as my main coding helper for our group’s **UAV semantic segmentation** work (UAVScenes AMtown02, interval=5). The job covered more than one notebook: matching images and masks, remapping label IDs, training, evaluation, leaderboard JSON, and GitHub setup.

**I used AI almost every day while I was actively building the project.** When I was only waiting for a long training run, I used it less, but whenever I was coding, debugging, or changing the pipeline, Cursor was part of my normal workflow.

The AI helped most with **structure and speed**: a PyTorch `Dataset`, the training loop, validation metrics, and plugging in **segmentation_models_pytorch** (UNet++ with a ResNet backbone). The **chat** was useful for stack traces and for edits that touched several files at once (`train.py`, `dataset.py`, `evaluate_submission.py`). Inline completion helped with small syntax issues and tensor shapes.

---

## Section 2: Understanding AI Limitations

AI was helpful, but it did not replace careful checking. Here are **real cases** from our project where the output was wrong or incomplete.

**1) GitHub push and SSH identity.** When we tried to push to my repo, Git returned *permission denied* and showed that GitHub saw a **different user** than `dingzhongwei`. The AI could explain SSH in general, but it could not see my key list or GitHub account settings. I fixed it by adding my own SSH key and updating `~/.ssh/config` so `github.com` used the correct `IdentityFile`. Without reading the **exact** error line, I might have kept repeating failed pushes.

**2) A Python `NameError` in evaluation code.** After we recovered some files, `evaluate_submission.py` crashed because the `if __name__ == "__main__"` block ran **before** helper functions like `_identity_transform` were defined. The AI had moved code in a way that looked fine at a glance, but **Python runs top to bottom**. I found it from the traceback. The fix was to move `main()` to the **end** of the file, after all helpers. This taught me: AI edits can break **definition order** if I do not reread the whole flow.

**3) Leaderboard scoring assumptions.** At first, both the AI and I treated the score like a simple weighted sum of raw Dice, mIoU, and FWIoU. The course actually used **normalized** terms in the final formula. The AI could not know the hidden rule until someone pasted the instructor’s description. I had to align my strategy with the real rule, not the generic one.

**4) Training on CPU for a long time.** The environment I started with effectively kept training on **CPU**. Runs were **very slow**. The AI suggested many training commands and loss ideas, but it **did not clearly tell me** to confirm that PyTorch was using the **GPU** first. I noticed the real problem myself when I opened **Task Manager** and saw the GPU barely used while CPU stayed busy. After that, I installed the right **CUDA** build of PyTorch and trained on GPU. That single change saved a huge amount of wall-clock time.

---

## Section 3: Engineering Validation

I did not want “pretty loss” on broken data. I used a few checks that matched real engineering work:

1. **Image–mask pairing:** same stem for `.jpg` and `.png`, and **nearest** resize for masks so labels were not blurred into wrong classes.  
2. **Train vs validation:** I watched validation **mIoU** and related metrics, not only training loss.  
3. **Metrics:** Dice, mIoU, and FWIoU from a confusion matrix, not only pixel accuracy.  
4. **Visual exports:** I saved **input / ground truth / prediction** panels for one frame to catch color and alignment bugs.  
5. **Hardware check:** after the CPU issue, I verified **GPU usage** in Task Manager or `nvidia-smi` before starting long jobs.

These steps caught shape issues for models that need sizes divisible by 16 or 32, and they stopped me from trusting one lucky number without context.

---

## Section 4: Problem-Solving Process

**Class imbalance** was a core difficulty. Background pixels dominated, and small classes were easy to ignore. We moved to **UNet++ with a pretrained encoder**, used **class-weighted cross-entropy**, and added a **Dice-style term** so the loss matched “overlap” style metrics better than plain pixel classification alone.

The biggest **accuracy** jump for us came from a **weighted ensemble** of several checkpoints. Each fold or push-round model had slightly different strengths. **Weighted assembly** means we averaged **logits** with positive weights (for example from different K-fold runs), then took `argmax`. The goal was to **combine strengths** instead of betting everything on one model. Finding good weights still needed experiments and comparison tables, not only AI guesses.

The biggest **time** problem was training on **CPU** for a long period because the stack I had at first was effectively CPU-oriented. The diagnosis was not magical: I watched **Task Manager**, saw the pattern, then switched to a **GPU + CUDA** setup. AI helped write code, but **I** had to connect the symptom (slow runs) to the cause (wrong device).

---

## Section 5: Learning Growth

I used to think segmentation was “download a model and train.” Now I think in terms of **labels, ignore pixels, class weights, input resolution, and metric definitions**. I am more comfortable with a small end-to-end path: **data → train → evaluate → ensemble → JSON**.

I also improved at **reading terminal errors** (SSH, `NameError`, shape errors) instead of blindly retrying. The course helped me link **what the UAV sees** to a usable map-like output, even though my own task was only the segmentation module.

---

## Section 6: Critical Reflection

AI clearly **speeds up** coding and explains APIs I had never used. It can also **hide blind spots**. In my case, it never pushed me early enough to ask: **“Is PyTorch actually using CUDA?”** I lost a lot of time on CPU training before I used Task Manager and fixed the environment. That is an important lesson: **some bottlenecks are not in the math—they are in the machine setup**, and I must verify them myself.

AI also tempted me, at times, to accept long answers without tracing **definition order** in Python or **identity** in Git. Going forward, I will keep two habits:  
(1) **Smoke-test on a tiny batch** and check device and data before long runs.  
(2) **Paste official rules** (grading, scoring) into the chat instead of assuming the model knows our course.

---

## Section 7: Evidence (Optional)

### 7.1 Definition order bug (concept)

Running `main()` before helper functions caused `NameError`. Fix: place `if __name__ == "__main__": main()` **after** helpers such as `_identity_transform`.

### 7.2 SSH error pattern (concept)

If the remote says access is denied to user **A** while you intend user **B**, treat it as a **credential mapping** problem, not random Git flags.

### 7.3 Useful prompt style

“Explain this traceback line by line and give three likely causes I can check in five minutes.” This reduced vague answers.


