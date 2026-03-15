# Deconstructing Karpathy's Autoresearch: A Line-by-Line Guide to Autonomous AI Training with Practical Examples

## Introduction: What is Autoresearch?

Imagine a research lab where the scientist never sleeps, never gets tired, and never stops experimenting. This scientist works with complete focus, running hundreds of experiments in a single night. Each experiment explores a different idea—a new neural network architecture, a tweaked hyperparameter, a different optimizer. When an experiment shows promise, the scientist keeps it and builds upon it. When it fails, the attempt is discarded and forgotten. By morning, the lab is filled with the evidence of a night of discovery: a clear path of improvements, tangible progress made, and a model that works better than it did before. This is not science fiction. This is what "autoresearch" means—an AI agent conducting autonomous research on itself, running experiments 24 hours a day and discovering improvements that even experienced human researchers might miss.

The fundamental idea is deceptively simple: give an AI agent a working piece of code—in this case, the training script for a large language model—and instruct it to make that code better by running experiments and learning from the results. The agent modifies the code, trains for a fixed time, measures whether performance improved, and then decides whether to keep the change or revert. It repeats this loop hundreds of times, each iteration building on the last successful improvement. The result is a cascade of small, additive changes that stack together into meaningful progress.

What makes this particularly powerful is that it targets a core bottleneck in artificial intelligence research. Training modern language models requires countless decisions about architecture, hyperparameters, optimizers, and training procedures. Each decision has been traditionally made by human researchers through intuition, paper-reading, and careful experimentation—a slow, expensive, and often frustrating process. A single training run can cost thousands of dollars in compute time and weeks of human attention tracking results, analyzing failures, and generating new ideas. Autoresearch automates this entire iterative cycle, allowing the AI to explore the space of possible improvements systematically and tirelessly. In the experiment described here, the agent ran approximately 700 experiments over two days. Out of these, about 20 represented genuine improvements that stacked together to produce an 11% better model.

The concrete result was a reduction in the "Time to GPT-2" leaderboard metric from 2.02 hours to 1.80 hours. This means that with the same computational resources, we can now train a GPT-2 level model nearly 22 minutes faster—a substantial gain in an area where every percentage point of efficiency matters. What's more, all of these improvements were additive and transferred successfully to larger models (from depth 8 to depth 24), proving their robustness. The discoveries included tangible, specific findings: the agent noticed that the existing QK normalization lacked a scaling multiplier, making attention too diffuse; it discovered that Value Embeddings—additional learned vectors added to the attention mechanism—benefited from regularization that was missing; it tuned the banded attention pattern that had been set too conservatively; it fixed suboptimal AdamW optimizer beta parameters; it adjusted the weight decay schedule; and it refined network initialization parameters. These are not vague or trivial changes—they address real design choices that human researchers would normally spend weeks investigating. According to the author, these were improvements that had been overlooked despite substantial manual tuning over a good amount of time.

The broader significance extends far beyond a single codebase. This experiment demonstrates a viable path toward fully autonomous AI research swarms. The architecture is minimal: three files covering data preparation, training code, and agent instructions. The human sets up the system and writes the initial program.md that guides the agent's behavior. Then the agent takes over, modifying the training script indefinitely while the human sleeps or focuses on other tasks. In principle, you could have multiple agents running with different prompts or on different model scales, collaborating or competing to accelerate progress. As the author notes, all frontier AI labs will eventually do this—it represents the "final boss battle" of AI engineering. The complexity increases at larger scales (you're not just tuning a single train.py file anymore), but the core idea scales: you spawn swarms of agents, have them tune smaller models first, promote the most promising ideas to larger scales, and let humans contribute on the fringes if desired. More generally, any metric that can be evaluated efficiently—whether it's model performance, code correctness, product metrics, or scientific outcomes—can be the target of an autonomous research agent. This opens the door to automated discovery across many domains.

The genius of this particular implementation lies in its deliberate simplicity. By constraining the agent to modify only a single file (train.py), the scope remains manageable and diffs are reviewable. By fixing the time budget to exactly 5 minutes for every experiment, all results become directly comparable regardless of architectural changes—the metric measures the best model achievable within a fixed computational investment, not final convergence quality. The git branch workflow provides a natural safety net: failed experiments are reverted automatically, while successful ones move the branch forward, creating a clean chain of improvements. And by optimizing for val_bpb (validation bits per byte), a metric that is vocabulary-size-independent, the agent can fairly compare changes that alter the tokenizer or model size.

Autoresearch is not about replacing human researchers. It's about removing the tedious, repetitive aspects of iterative optimization—the thousand small adjustments, the constant trial and error, the waiting for runs to finish—so that human researchers can focus on higher-level strategy, novel directions, and creative leaps. It's about democratizing capability: a small team with limited compute can now harness an AI agent to perform the kind of exhaustive hyperparameter search that previously required large, well-funded labs with legions of engineers. It's about accelerating progress across the board: faster iterations mean ideas move from conception to validation more quickly, which compounds into faster overall advancement.

This matters because the implications reach beyond language models. If AI agents can autonomously improve their own training procedures, they can similarly improve other software systems, optimize manufacturing processes, refine scientific experiments, or iterate on product designs. The pattern is general: an agent that can read code, propose modifications, evaluate outcomes, and learn from feedback can automate improvement in any domain with a measurable objective and a way to try changes safely. Today, that's possible for model training because the evaluation is fast (5 minutes), the code is self-contained, and the metric is clear. Tomorrow, it could apply to many other technical workflows.

In a sense, autoresearch marks the beginning of a new era in scientific and engineering discovery—one where AI systems don't just assist human researchers but become researchers themselves, conducting long-term, open-ended experiments that compound over time. The vision put forward by the author is striking: a future where"research is now entirely the domain of autonomous swarms of AI agents running across compute cluster megastructures in the skies," iterating through generations of code without direct human intervention. That future may still be on the horizon, but this project shows that we are already taking concrete steps toward it. We are witnessing the first moments of a transition from human-directed experimentation to autonomous scientific discovery—a shift that could reshape not just how we build AI, but how we acquire knowledge across many fields. The robot chef analogy captures the essence: set the goal, provide the tools and ingredients, and let the system explore overnight. When you return, you'll find not just a better recipe, but a new understanding of what's possible.

### Who is Andrej Karpathy?

You might wonder who came up with this robot chef idea. Andrej Karpathy is a computer scientist who has spent many years working with artificial intelligence. He studied at Stanford University, one of the top places for computer science research, where he learned about deep learning - the technology that makes modern AI possible. After Stanford, he went to work at Tesla, the electric car company. At Tesla, he led the team that built Autopilot, the system that helps cars steer themselves, brake when needed, and stay in their lane. This was one of the first times that AI technology was put into a product that regular people could buy and use in their cars.

Later, he became one of the early workers at OpenAI, the research company that created the GPT series of language models including ChatGPT. He was part of the original team that helped build the foundation for today's AI boom. Even though he eventually moved on to other projects, his work helped shape how the whole AI industry developed. Today he is well-known for his ability to explain complicated AI topics in ways that anyone can understand. He makes videos and writes articles about how AI systems work. People listen to what he has to say because he combines deep technical knowledge with a real talent for teaching.

His "autoresearch" project continues this tradition of making complex ideas accessible and practical. The project tackles a fundamental challenge: training AI models is both expensive and slow. If an AI system can help improve its own training code, we could speed up progress across the entire field of artificial intelligence. Karpathy's reputation gives credibility to this research direction. More importantly, his track record shows he focuses on problems that actually matter in the real world, not just theoretical puzzles. He is not just someone who writes papers - he has built systems that affect millions of people through Tesla's Autopilot. That practical experience makes his work particularly valuable to pay attention to.

### What Are These Technologies?

To understand why autoresearch is exciting, we need to understand what happens when people build AI like ChatGPT. Building a large language model involves making thousands of small decisions. How many layers should the AI brain have? What learning rate works best? Should we use one type of attention mechanism or another? For years, human researchers have had to spend months or even years experimenting with these settings. Each experiment can cost thousands or millions of dollars in computer time because training these models requires huge amounts of computing power.

Large Language Models (LLMs) - sometimes called GPT models because GPT stands for Generative Pre-trained Transformer - are AI systems that learn from reading vast amounts of text. They work by trying to predict what word comes next in a sentence. By practicing this over and over with billions of examples, they learn to generate realistic text, answer questions, write computer code, and even mimic different writing styles. These models have become the foundation for most modern AI assistants.

At the heart of these systems is something called a neural network. Think of a neural network like a team of many tiny learners working together. Each learner picks up on a small piece of the puzzle. When you connect thousands of these learners, they can recognize very complex patterns. The network starts knowing nothing at all. It makes random guesses at first. Then it practices, over and over, with millions of examples. This practice process is what researchers call training. Training is like practicing piano scales. You play the same exercises many times. Each time you make small adjustments to your finger positions. Gradually, your fingers learn the right movements and you can play smoothly without thinking.

But training needs rules, just like practicing piano needs guidance. How long should you practice each day? How fast should you go through the exercises? These settings are called hyperparameters. They are like the temperature and baking time you set when making bread. If the oven is too hot, the bread burns. If it is too cool, the bread does not bake properly. Hyperparameters control how fast the AI learns, how much it remembers, and many other details about its practice sessions. Finding good hyperparameters makes the difference between a system that learns effectively and one that never really improves.

Finally, there is optimization. This is the coach who watches your piano practice and suggests changes. Maybe you need to slow down in certain sections. Maybe you should use a different finger for a tricky passage. The optimizer is that coach for the neural network. After each practice round, the optimizer calculates how to adjust the thousands or millions of tiny settings inside the network to reduce future mistakes. It guides the learning process toward better performance.

All these pieces fit together: the GPT architecture provides the overall brain structure, the neural network is the team of learners, training is the practice process, hyperparameters are the practice settings, and optimization is the coaching feedback. When we get all these elements working well together, we get a powerful AI system.

This project might seem like just a clever technical experiment, but it points to a much bigger shift in how science happens. For the first time, AI systems can conduct meaningful research on themselves, running experiments 24 hours a day and discovering improvements that even experienced human researchers might not think of.

### Why Should Anyone Care?

You might wonder why we should care about automated AI research. The answer is that faster AI research could help solve some of humanity's biggest challenges. Think about scientists who search for new medicines. They often have to test thousands or even millions of possible drug combinations. This process can take decades. If AI models that help with this research become better and faster, we could discover life-saving treatments much more quickly. Better AI means better tools for doctors. Imagine an AI that helps read medical scans with superhuman accuracy, spotting tumors or other problems that humans might miss. Or an AI that predicts which patients need urgent care so doctors can prioritize effectively. These systems need constant improvement to reach their full potential. Autonomous research could make those improvements happen faster than ever before.

Consider education. Right now, many students use basic tutoring software that follows fixed programs. But what if the AI behind these tools could learn more effectively on its own? It could adapt to each student's unique learning style, helping children understand difficult concepts in math, reading, or science in ways that work specifically for them. The same goes for climate science. Predicting weather patterns and long-term climate change requires enormous computing power. More efficient AI models could run on smaller computers, bringing advanced forecasting capabilities to communities and countries that currently lack access to supercomputers.

Even everyday products could become better and cheaper. Better AI means companies can design products with less waste, optimize delivery routes to save fuel, and offer more personalized services at lower cost. The impact is not about making smarter robots that walk around. It is about using technology to solve real human problems. Autonomous AI research is like giving scientists a tireless assistant who can also improve its own skills. That assistant could help us tackle some of the world's biggest challenges, from health to environment to education.

This matters because it could dramatically speed up the pace of AI development. Powerful models that might have taken years to develop could arrive in months or weeks. It also democratizes research: small teams with limited resources can let AI agents do the tedious trial-and-error work that normally requires large, well-funded research labs. Today, this is possible because language models have become remarkably skilled at writing code and reasoning through problems. Open-source tools make experimentation accessible, and a single powerful computer (called a GPU) can run these experiments. This is part of a broader trend toward automation of scientific discovery, where AI does not just assist humans but actively conducts research. As we will see, the agent discovered real, tangible improvements that stacked together to produce an 11% better model. This is not a distant dream - it is happening right now.

## The Big Picture: How Does It Work?

Let me explain how this autonomous loop using familiar analogies. Think of training a neural network like teaching a student to recognize cats in pictures. You start with a blank slate, show it examples, correct its mistakes, and gradually it learns. But you also have to decide how fast the student learns (learning rate), how many practice sessions it needs (epochs), and what teaching methods work best (optimizer choice). These decisions are hyperparameters, and finding good ones is more art than science.

### Everyday Analogies for Key Concepts

**Neural networks as students studying:** Picture a student learning a new subject. At first they know nothing. They read textbooks, attend lectures, and take practice quizzes. Each time they make a mistake, they adjust their thinking. Over time they get better. A neural network works the same way. It starts with random connections, like a confused student. It processes examples, sees where it went wrong, and tweaks its internal structure. The network gradually builds understanding, not by memorizing facts, but by recognizing patterns. This analogy reminds us that learning takes time and repetition.

**Training as practicing piano:** Learning piano requires daily practice. You start with simple scales, then move to short pieces, then concertos. Each practice session strengthens muscle memory. Similarly, an AI model practices with many data examples. It plays the same "scales" of prediction many times. Over time, its accuracy improves. The parallel reminds us that skill development is gradual and requires consistent effort.

**Attention mechanism as reading a book with a highlighter:** When you read an important document, you do not treat every word equally. You naturally focus on headings, key sentences, and numbers. Your attention goes to what matters most for understanding the main points. The attention mechanism in AI works similarly. It allows the model to weigh the importance of different words when making predictions. When the model reads "The cat sat on the mat because it was tired", the attention mechanism helps the model figure out that "it" refers to the cat, not the mat. This mirrors how humans prioritize information while reading.

**Optimizers as coaches adjusting training methods:** A good sports coach watches an athlete's performance and suggests changes. Maybe they need more cardio training, or a different diet, or a technical tweak to their form. The optimizer is exactly that coach for the neural network. After each practice round, the optimizer looks at what happened and calculates how to adjust the network's internal settings to reduce future mistakes. It directs the whole learning process toward better performance.

**Hyperparameters as recipe settings:** Making the perfect soup requires setting the right stove temperature, the perfect simmer time, and the exact amount of salt. Change one of these settings and the result is different. Hyperparameters are exactly those oven temperature and cooking time settings, but for AI training. They control how fast the model learns, how much information it keeps in memory, and many other details about its practice sessions. Getting them just right can be the difference between a model that learns quickly and effectively and one that never really improves. This is why searching for good hyperparameters is so important and so time-consuming.

To make this accessible, let's clarify some core concepts more formally.

A **neural network** is a type of computer program loosely modeled on the brain. It is built from many small computing units called neurons, organized in layers. Each connection between neurons has a number (a weight) that determines how much influence one neuron has on another. During training, these weights get adjusted so the network's outputs become more accurate. More layers generally mean the network can learn more complex patterns, but also require more computation and data.

**Training** is the learning process itself. You feed the network many examples (like pictures of cats and not-cats), it makes predictions, measures how wrong it is using a loss function, and then updates its weights to reduce that wrongness. This cycle repeats millions of times. Think of it like practicing a skill over and over, gradually getting better.

**Hyperparameters** are the settings you choose before training starts, the knobs and dials that control how training proceeds. They include the learning rate (how big each weight update should be, like the size of your steps when learning), the batch size (how many examples to process before updating weights), the optimizer (the algorithm that decides how to update weights based on gradients), the number of layers, and many others. Finding good hyperparameters is a mix of science and art; small changes can have big effects, and what works for one model might not work for another. That is why automated search like this project is valuable.

**Optimization** refers to the mathematical methods used to adjust the weights during training. The optimizer looks at the gradients (which indicate which direction reduces loss) and decides how exactly to move each weight. Different optimizers, like SGD, Adam, or the Muon used here, have different strategies for combining past information, adapting step sizes, and preventing overshooting. Good optimization makes training faster and more stable.

Autoresearch automates this decision-making process. Here is how the loop works in plain language:

The AI agent starts with a working version of the training code. It runs an experiment for exactly 5 minutes, measuring how well the model performs on a validation set using a metric called "val_bpb" (validation bits per byte). Lower val_bpb means the model is better at predicting text.

These analogies help us understand the pieces of AI training. Now let's look at how they work together in an automated system.

| AI Concept | Everyday Analogy | Simple Explanation |
|------------|------------------|-------------------|
| Neural network | Student studying | Many small units working together to recognize patterns |
| Training | Practicing piano | Repeated exercises that gradually improve skill |
| Attention mechanism | Reading with a highlighter | Focusing on the most important information |
| Optimizer | Coach | Provides feedback and adjustments to improve |
| Hyperparameters | Recipe settings | Controls like temperature and time that affect the outcome |

### Understanding the Metrics: Bits per Byte and Validation Loss

How do we know if an AI model is actually good? We need ways to measure its performance, just like teachers use tests to see if students are learning the material. In AI research, two common measures are called "bits per byte" and "validation loss". These might sound technical, but the ideas behind them are quite simple.

Think about bits per byte like this: The AI model reads some text and tries to predict what word comes next. After you show it the actual word, you can ask: how surprised was the model? If it predicted the word perfectly, it was not surprised at all. If its prediction was completely wrong, it was very surprised. Bits per byte is a number that adds up all that surprise across many thousands of words. A lower number means the model was less surprised - meaning it made fewer mistakes in its predictions.

Here is an everyday analogy: imagine a spelling test. A student who spells almost every word correctly gets a high grade (like an A). A student who makes many spelling errors gets a lower grade (like a C or D). Bits per byte is like that grade, but for an AI taking a massive language test with millions of words. It tells us how well the model understands language patterns.

The other important measure is "validation loss". This checks whether the model really learned or just memorized. During training, the model sees a set of practice problems - this is called the training set. But researchers also give the model a separate set of problems it has never seen before. That is the validation set. The model's performance on these new, unseen problems is its validation loss. A low validation loss means the model can apply what it learned to new situations it has not encountered before.

This is exactly like a student who studies specific example problems for an exam. If the exam questions are slightly different from the practice problems but still test the same concepts, a student who truly understands the material will still do well. If the student just memorized the practice answers without understanding, they will struggle with the new problems. Validation loss tells us if the AI is truly understanding patterns and concepts, or if it is just memorizing answers without generalizing. Both metrics guide researchers toward better models - they are the feedback signals that indicate whether the AI is improving or getting worse.

### The 5-Minute Experiment Loop

Now let's look at how the autonomous system actually runs experiments. Here is the step-by-step process that repeats over and over:

**Step 1: Start with a working version.** The AI agent begins with a version of the training code that we know works - either the original code or the best version found so far.

**Step 2: Run a 5-minute training experiment.** The agent starts the training program and lets it run for exactly 5 minutes of actual training time (not counting startup and compilation). During this time, a neural network model trains from scratch or continues training, trying to learn from the data.

**Step 3: Measure the results.** When the 5 minutes are up, the program stops and prints out some key numbers, especially the val_bpb. This tells us how good the model became after exactly 5 minutes of training with the current settings.

**Step 4: Record the experiment.** The agent saves information about what was tried and what result it produced. This creates a history of all experiments, both successful and unsuccessful.

**Step 5: Modify the code intelligently.** This is where the agent's creativity comes in. It reads the current training code and makes a change - maybe it increases the learning rate, adds a new type of normalization layer, changes the optimizer settings, or adjusts the model architecture by adding or removing layers.

**Step 6: Save the change with git.** The agent saves this modified version of the code using a system called git. Git is like a time machine for code - it keeps track of every change made and allows you to go back to previous versions. The agent puts this change on a special branch (more on branches below).

**Step 7: Test the modified version.** The agent runs another 5-minute experiment with the modified code to see if the change made things better or worse.

**Step 8: Keep or discard.** When this new experiment finishes, the agent compares the new val_bpb to the previous best. If the new number is lower (better), the agent keeps this change - this becomes the new best version, and the branch moves forward. If the new number is higher (worse) or about the same, the agent discards the change and goes back to the previous version (the one with the better val_bpb) before trying something else.

**Step 9: Repeat forever.** This cycle repeats, hundreds of times. The agent is not just randomly changing things. It looks at the history of what worked and what failed, reads the code to understand what each change does, and plans the next experiment based on patterns it observes. Over 2 days, with approximately 100 experiments (each 5 minutes), the agent discovered about 20 real improvements that stacked together to produce an 11% better model.

The genius of this design is the fixed 5-minute time budget. Every experiment runs for exactly 5 minutes, regardless of what changes the agent makes. This means all results are directly comparable. If the agent makes the model bigger (which normally would take longer to train), it still stops after 5 minutes. So we are measuring which model configuration makes the fastest progress during that fixed time. We are not comparing the final performance after training to completion, but rather the performance achieved within exactly the same time investment. This makes the search meaningful: we want to know what the best model is that we can get after investing precisely 5 minutes of training time.

### Git Branches: The Time Machine for Code

You might have noticed the term "git branch" in the description above. Git is a system that developers use to keep track of changes in code. Think of git like a time machine or a save point in a video game. At any moment, you can save the exact state of your code. Later, if you try something that does not work, you can go back to a previous saved state.

A "branch" in git is like an alternate timeline that branches off from the main line of development. Imagine you are writing a story. You have written up to chapter 10, and this is your "main branch" or "master branch." Now you want to try a different direction for the story. You create a new branch called "experiment-1." This branch starts exactly at chapter 10, but now you can write different chapters along this branch. If you like the new direction, you can merge it back into the main story. If you do not like it, you can simply abandon the branch and go back to your original timeline at chapter 10, unchanged.

In autoresearch, the agent creates a branch specifically for its experiments - something like "autoresearch/march15" (using the date). This branch starts from a known good version of the code. Every time the agent tries a change, it commits that change to this experimental branch. If the experiment improves the model, the branch moves forward - this new improved code becomes the starting point for the next experiment. If the experiment makes things worse, the agent goes back to the previous commit on the branch, effectively erasing the failed experiment and trying something new from the same starting point.

This branch system is crucial because it allows the agent to experiment freely without losing progress. The agent can try wild ideas, knowing that if they fail completely, it can always go back to the last good version. Over time, the branch accumulates only the successful changes, creating a chain of improvements that build on each other. When you look at the git history later, you can see exactly which changes were tried and which ones actually helped.

### Comparing Experiments: What Gets Kept?

The agent runs experiment after experiment, but not all experiments are treated equally. The decision about whether to keep a change is based on the val_bpb metric. Here is how the comparison works:

The agent always compares the new experiment's val_bpb to the current best val_bpb among all experiments that have been kept so far. A "kept" experiment is one that showed improvement and whose code is now the current baseline.

If the new experiment's val_bpb is lower than the current best - meaning the model achieved better performance - then this change is declared a success. The agent keeps this new version of the code. From now on, this becomes the new baseline. The git branch stays at this commit and all future experiments will build upon this improved code.

If the new experiment's val_bpb is higher than the current best - meaning performance got worse - then the change is discarded. The agent resets the code back to the previous version (the one with the better val_bpb). The experiment is still recorded in the results log so the agent knows not to try that exact change again, but the code itself goes back to the last good state.

If the new experiment's val_bpb is exactly the same as the current best, this is usually considered a discard as well. The agent tries to avoid keeping changes that do not improve the metric, because they add complexity without benefit. There is one exception: if the change simplifies the code while maintaining performance, that might be worth keeping even without improvement, because simpler code is easier to understand and less likely to break later.

This creates what is essentially an automated hill-climbing process. Each successful change moves the code to a slightly higher position on the hill (lower val_bpb means we are going "up" toward better performance). Failed experiments are like taking a step in the wrong direction - we go back to where we were and try a different direction. After hundreds of these attempts, the agent slowly but steadily climbs toward better performance.

The fact that the agent discovered about 20 improvements that stacked together shows that the changes are generally independent and additive - you can make one improvement, then another on top of that, then another on top of that, each time getting a better result. This is not always true in AI research; sometimes changing one thing breaks another. But in this case, the improvements were largely orthogonal, allowing them to combine into a significantly better final model.

## The Three Files: Understanding the Architecture

The entire autoresearch system consists of three files, each with a clear and distinct purpose. Think of them as the foundation, the workbench, and the instruction manual.

The first file is `prepare.py`. This is the foundation that never changes. It handles all the one-time setup: downloading the training data from the internet, training a tokenizer that breaks text into pieces the model can understand, and providing utility functions for loading data and evaluating the model. Once you run this file, it creates a cache directory with all the necessary data and the tokenizer. The AI agent is not allowed to touch this file. It is read-only infrastructure.

The second file is `train.py`. This is the workbench where all the action happens. This single file contains the complete definition of the GPT model, the optimizer that trains it, the hyperparameters that control its behavior, and the training loop that runs for 5 minutes. This entire 630-line file is what the AI agent modifies. The agent can change anything: the model architecture, the optimizer settings, the learning rates, the batch size, everything. Every experiment potentially produces a different version of this file.

The third file is `program.md`. This is the instruction manual for the AI agent. The human researcher writes and iterates on this file. It contains the step-by-step instructions for how the agent should conduct its autonomous research. It explains the setup process, what the agent can and cannot modify, what metric to optimize for, how to log results, and the endless loop of experiment, evaluate, keep or discard. The human might improve these instructions over time to make the agent more effective at finding improvements.

When you put these three files together, you have a complete autonomous research system. The human sets up the data once with prepare.py, writes the initial instructions in program.md, and then lets the AI agent loose on train.py. The agent runs experiments, modifies the code, tests the modifications, and accumulates successful changes. You wake up in the morning to a git repository full of commits and a results file showing all the discoveries made overnight.

## Deep Dive: prepare.py - The Foundation That Never Changes

Let me walk you through `prepare.py` line by line, explaining what each part does in simple terms. This file is like the kitchen setup before cooking begins.

```python
"""
One-time data preparation for autoresearch experiments.
Downloads data shards and trains a BPE tokenizer.

Usage:
    python prepare.py                  # full prep (download + tokenizer)
    python prepare.py --num-shards 8   # download only 8 shards (for testing)

Data and tokenizer are stored in ~/.cache/autoresearch/.
"""
```

At the top, we have a docstring, which is just a comment that explains what this file does. Anyone reading this code immediately understands its purpose. The file handles one-time preparation: downloading data and training a tokenizer. It tells us that data gets stored in a hidden folder in the user's home directory called `.cache/autoresearch/`.

```python
MAX_SEQ_LEN = 2048       # context length
TIME_BUDGET = 300        # training time budget in seconds (5 minutes)
EVAL_TOKENS = 40 * 524288  # number of tokens for val eval
```

These are constants, values that stay fixed throughout the entire autoresearch process. MAX_SEQ_LEN is 2048, which means the model looks at sequences of 2048 tokens at a time. Think of this as the model's working memory. TIME_BUDGET is 300 seconds, which is exactly 5 minutes. Every training experiment stops after this time. EVAL_TOKENS determines how much data we use to evaluate whether the model is good; it uses 20 million tokens for validation.

```python
CACHE_DIR = os.path.join(os.path.expanduser("~"), ".cache", "autoresearch")
DATA_DIR = os.path.join(CACHE_DIR, "data")
TOKENIZER_DIR = os.path.join(CACHE_DIR, "tokenizer")
BASE_URL = "https://huggingface.co/datasets/karpathy/climbmix-400b-shuffle/resolve/main"
MAX_SHARD = 6542
VAL_SHARD = MAX_SHARD
VOCAB_SIZE = 8192
```

Here we define where files live on disk. CACHE_DIR points to `~/.cache/autoresearch/` on any operating system. Inside that, DATA_DIR holds downloaded data shards, and TOKENIZER_DIR will store the trained tokenizer. BASE_URL tells us where to download the training data from, it is a specific dataset called "climbmix-400b-shuffle" hosted on Hugging Face, which is a popular platform for sharing machine learning datasets. MAX_SHARD is 6542, meaning there are 6543 shard files numbered from 0 to 6542. We designate the last shard as the validation shard, which we use to evaluate model performance. VOCAB_SIZE is 8192, meaning our tokenizer will learn 8192 different tokens (pieces of words or characters).

```python
SPLIT_PATTERN = r"""'(?i:[sdmt]|ll|ve|re)|[^\r\n\p{L}\p{N}]?+\p{L}+|\p{N}{1,2}| ?[^\s\p{L}\p{N}]++[\r\n]*|\s*[\r\n]|\s+(?!\S)|\s+"""
SPECIAL_TOKENS = [f"<|reserved_{i}|>" for i in range(4)]
BOS_TOKEN = "<|reserved_0|>"
```

These control how text gets broken into tokens. SPLIT_PATTERN is a regular expression that determines how words are split. It is quite complex but basically follows the GPT-4 style of splitting, which handles punctuation, numbers, and whitespace intelligently. SPECIAL_TOKENS defines four special tokens, and BOS_TOKEN is the "beginning of sequence" token that marks the start of each training example. This token helps the model know when a new piece of text begins.

Now let's look at the data download functions:

```python
def download_single_shard(index):
    """Download one parquet shard with retries. Returns True on success."""
    filename = f"shard_{index:05d}.parquet"
    filepath = os.path.join(DATA_DIR, filename)
    if os.path.exists(filepath):
        return True

    url = f"{BASE_URL}/{filename}"
    max_attempts = 5
    for attempt in range(1, max_attempts + 1):
        try:
            response = requests.get(url, stream=True, timeout=30)
            response.raise_for_status()
            temp_path = filepath + ".tmp"
            with open(temp_path, "wb") as f:
                for chunk in response.iter_content(chunk_size=1024 * 1024):
                    if chunk:
                        f.write(chunk)
            os.rename(temp_path, filepath)
            print(f"  Downloaded {filename}")
            return True
        except (requests.RequestException, IOError) as e:
            print(f"  Attempt {attempt}/{max_attempts} failed for {filename}: {e}")
            for path in [filepath + ".tmp", filepath]:
                if os.path.exists(path):
                    try:
                        os.remove(path)
                    except OSError:
                        pass
            if attempt < max_attempts:
                time.sleep(2 ** attempt)
    return False
```

This function downloads one data shard. Let's break it down. It constructs the filename like `shard_00042.parquet` using zero-padded numbers. It checks if the file already exists, and if so, returns True immediately because we do not need to download it again.

If we need to download it, the function constructs the full URL and attempts the download up to 5 times. The download streams the file in 1-megabyte chunks to avoid memory issues. It writes to a temporary file first (`filepath + ".tmp"`) and only renames it to the final filename when the download completes successfully. This prevents corrupted partial downloads from being mistaken for complete files.

If an attempt fails, it cleans up any temporary or partial files and waits with exponential backoff, 2 seconds, then 4, then 8, and so on. This is polite to the server and gives transient network issues time to resolve.

```python
def download_data(num_shards, download_workers=8):
    """Download training shards + pinned validation shard."""
    os.makedirs(DATA_DIR, exist_ok=True)
    num_train = min(num_shards, MAX_SHARD)
    ids = list(range(num_train))
    if VAL_SHARD not in ids:
        ids.append(VAL_SHARD)

    existing = sum(1 for i in ids if os.path.exists(os.path.join(DATA_DIR, f"shard_{i:05d}.parquet")))
    if existing == len(ids):
        print(f"Data: all {len(ids)} shards already downloaded at {DATA_DIR}")
        return

    needed = len(ids) - existing
    print(f"Data: downloading {needed} shards ({existing} already exist)...")

    workers = max(1, min(download_workers, needed))
    with Pool(processes=workers) as pool:
        results = pool.map(download_single_shard, ids)

    ok = sum(1 for r in results if r)
    print(f"Data: {ok}/{len(ids)} shards ready at {DATA_DIR}")
```

This function downloads multiple shards in parallel. It creates the data directory if needed, then determines which shards need to be downloaded. It always includes the validation shard (6542) even if the user only asked for a subset of training shards. It counts existing files to avoid re-downloading, then uses a pool of worker processes to download multiple shards simultaneously. Parallel downloading speeds up the process significantly, especially for slow networks.

Now the tokenizer training:

```python
def train_tokenizer():
    """Train BPE tokenizer using rustbpe, save as tiktoken pickle."""
    tokenizer_pkl = os.path.join(TOKENIZER_DIR, "tokenizer.pkl")
    token_bytes_path = os.path.join(TOKENIZER_DIR, "token_bytes.pt")

    if os.path.exists(tokenizer_pkl) and os.path.exists(token_bytes_path):
        print(f"Tokenizer: already trained at {TOKENIZER_DIR}")
        return

    os.makedirs(TOKENIZER_DIR, exist_ok=True)

    parquet_files = list_parquet_files()
    if len(parquet_files) < 2:
        print("Tokenizer: need at least 2 data shards (1 train + 1 val). Download more data first.")
        sys.exit(1)
```

This function checks if we already have a trained tokenizer by looking for two files: `tokenizer.pkl` (the actual tokenizer) and `token_bytes.pt` (a lookup table we'll explain later). If both exist, it returns immediately. Otherwise, it creates the tokenizer directory and checks that we have at least 2 data shards, one for training and one for validation.

```python
    tokenizer = rustbpe.Tokenizer()
    vocab_size_no_special = VOCAB_SIZE - len(SPECIAL_TOKENS)
    tokenizer.train_from_iterator(text_iterator(), vocab_size_no_special, pattern=SPLIT_PATTERN)
```

Here we create a BPE tokenizer using the `rustbpe` library, which is a fast implementation written in Rust. BPE stands for Byte Pair Encoding, a method for building a vocabulary of tokens. The algorithm starts with individual characters and repeatedly merges the most frequent pairs until we reach the desired vocabulary size. The `train_from_iterator` method reads text from the `text_iterator()` function and learns merges. We reserve 4 special tokens, so the actual learned vocabulary size is 8192 minus 4 equals 8188.

```python
    pattern = tokenizer.get_pattern()
    mergeable_ranks = {bytes(k): v for k, v in tokenizer.get_mergeable_ranks()}
    tokens_offset = len(mergeable_ranks)
    special_tokens = {name: tokens_offset + i for i, name in enumerate(SPECIAL_TOKENS)}
    enc = tiktoken.Encoding(
        name="rustbpe",
        pat_str=pattern,
        mergeable_ranks=mergeable_ranks,
        special_tokens=special_tokens,
    )
```

After training, we need to convert the rustbpe tokenizer into a format that OpenAI's `tiktoken` library can use. This is because tiktoken is widely used and well-optimized for tokenization. We extract the pattern (the splitting rules), the mergeable ranks (the learned token pairs), and add our special tokens with IDs that come after all the normal tokens. The `tiktoken.Encoding` object represents a complete tokenizer that we can save.

```python
    with open(tokenizer_pkl, "wb") as f:
        pickle.dump(enc, f)

    t1 = time.time()
    print(f"Tokenizer: trained in {t1 - t0:.1f}s, saved to {tokenizer_pkl}")
```

We save the tokenizer using Python's pickle module, which serializes the object to disk. Later, `train.py` will load this pickle file to get the tokenizer.

```python
    token_bytes_list = []
    for token_id in range(enc.n_vocab):
        token_str = enc.decode([token_id])
        if token_str in special_set:
            token_bytes_list.append(0)
        else:
            token_bytes_list.append(len(token_str.encode("utf-8")))
    token_bytes_tensor = torch.tensor(token_bytes_list, dtype=torch.int32)
    torch.save(token_bytes_tensor, token_bytes_path)
```

This section creates the `token_bytes.pt` file. For each token in the vocabulary, we compute how many bytes it takes to represent that token as UTF-8 text. Special tokens get 0 bytes because they do not represent actual text. This lookup table is crucial for the evaluation metric. Bits per byte (BPB) measures how many bits of information the model needs to predict each actual byte of text. When computing the loss, we exclude special tokens from both the numerator and denominator. Having this precomputed lookup makes evaluation efficient.

The training and validation data comes from Parquet files, which is a columnar storage format efficient for reading large datasets. The `text_iterator()` function yields documents from the training shards, and `make_dataloader()` creates an infinite stream of tokenized batches with a clever best-fit packing algorithm that minimizes padding and maximizes GPU utilization. The key insight is that documents have different lengths, and we want to pack them into fixed-length sequences (2048 tokens) without waste. The algorithm maintains a buffer of documents and for each row of the batch, it tries to fill the remaining space with the largest document that fits. If no document fits, it crops the shortest document to fill exactly the remaining space. This achieves 100% utilization with no padding.

The `evaluate_bpb()` function is the fixed evaluation metric that the agent cannot change. It runs the model on the validation set without updating weights, computes the cross-entropy loss (in nats, which are natural logarithm units), multiplies by the byte lengths of the target tokens, and converts to bits per byte by dividing by log(2). This metric is vocabulary-size-independent, which is important because the agent might change the vocabulary size in theory, and this ensures comparisons remain fair.

## Deep Dive: train.py - The File the AI Agent Modifies

The `train.py` file is where the magic happens. This 630-line file contains everything needed to train a GPT model, and the AI agent is free to modify any part of it. Let me walk through it section by section, explaining each concept in simple terms.

```python
"""
Autoresearch pretraining script. Single-GPU, single-file.
Cherry-picked and simplified from nanochat.
Usage: uv run train.py
"""
```

The docstring explains this is a simplified version of a larger project called nanochat. It is designed for single GPU training, making it accessible without complex distributed computing setups.

```python
import os
os.environ["PYTORCH_ALLOC_CONF"] = "expandable_segments:True"
os.environ["HF_HUB_DISABLE_PROGRESS_BARS"] = "1"

import gc
import math
import time
from dataclasses import dataclass, asdict

import torch
import torch.nn as nn
import torch.nn.functional as F

from kernels import get_kernel
cap = torch.cuda.get_device_capability()
repo = "varunneal/flash-attention-3" if cap == (9, 0) else "kernels-community/flash-attn3"
fa3 = get_kernel(repo).flash_attn_interface
```

The imports set up the environment. The first two lines set environment variables for PyTorch to use expandable memory segments (helps avoid fragmentation) and to disable Hugging Face progress bars (to keep logs clean). Then we import standard libraries: `gc` for garbage collection, `math` for math operations, `time` for timing, and `dataclass` for convenient configuration classes.

The PyTorch imports are the core neural network framework. We use `torch.nn` for neural network modules and `torch.nn.functional` for functions like activation functions.

The next few lines are important: they load a special kernel called Flash Attention 3. Attention is the key operation in transformer models, and Flash Attention is an optimized implementation that runs much faster by using GPU memory more efficiently. The code checks the GPU capability; if it is an H100 (capability 9.0), it uses one repository, otherwise it uses a community version that works on other GPUs. This abstraction means the agent does not have to worry about the low-level CUDA details, the kernel is loaded and available.

```python
from prepare import MAX_SEQ_LEN, TIME_BUDGET, Tokenizer, make_dataloader, evaluate_bpb
```

Here we import the fixed components from `prepare.py`: the constants and functions that the training script uses. The agent can modify everything else, but these are off-limits.

Now we come to the model configuration:

```python
@dataclass
class GPTConfig:
    sequence_len: int = 2048
    vocab_size: int = 32768
    n_layer: int = 12
    n_head: int = 6
    n_kv_head: int = 6
    n_embd: int = 768
    window_pattern: str = "SSSL"
```

A dataclass is a convenient Python class that automatically generates boilerplate code like initialization and representation. The `GPTConfig` class holds all the architectural hyperparameters that define the model structure. `sequence_len` is the context window (2048 tokens). `vocab_size` is the size of the token vocabulary, default is 32768, though the tokenizer actually uses 8192; note the discrepancy, but the code below will overwrite this with the actual tokenizer vocab size. `n_layer` is the number of transformer blocks (12). `n_head` is the number of attention heads (6). `n_kv_head` is the number of key-value heads (6), this differs from regular GPT where they would be the same; this allows for grouped query attention which is more efficient. `n_embd` is the embedding dimension (768), which is the size of the hidden representations throughout the model. `window_pattern` is "SSSL" which determines sliding window attention patterns, S means short window (half context), L means long window (full context), alternating SSSL means first three layers use short window, last layer uses long.

Now the building blocks:

```python
def norm(x):
    return F.rms_norm(x, (x.size(-1),))
```

This is a normalization function. Normalization is a technique that stabilizes neural network training by keeping activation values in a reasonable range. This particular function uses RMSNorm (Root Mean Square Normalization), which is simpler than LayerNorm and has become popular in recent models. It normalizes each vector independently by dividing by its root mean square.

```python
def has_ve(layer_idx, n_layer):
    """Returns True if layer should have Value Embedding (alternating, last always included)."""
    return layer_idx % 2 == (n_layer - 1) % 2
```

This helper function determines which layers get Value Embeddings, which are an advanced feature from the ResFormer paper. Value Embeddings are additional embeddings added to the attention values. The pattern is alternating: if we have an even number of layers, the decision alternates between even and odd indices. The last layer always gets one. This function implements that pattern concisely.

```python
def apply_rotary_emb(x, cos, sin):
    assert x.ndim == 4
    d = x.shape[3] // 2
    x1, x2 = x[..., :d], x[..., d:]
    y1 = x1 * cos + x2 * sin
    y2 = x1 * (-sin) + x2 * cos
    return torch.cat([y1, y2], 3)
```

This function applies Rotary Position Embeddings (RoPE). Position embeddings give the model information about where tokens appear in the sequence. Traditional positional embeddings are added directly to token embeddings. Rotary embeddings use a rotation in the vector space to encode position information in a way that is more computationally efficient and extrapolates to longer sequences. The math here rotates the query and key vectors by amounts determined by the precomputed cosine and sine values for each position. The operation splits the last dimension in half, applies the rotation formula, and concatenates back.

Now the core attention mechanism:

```python
class CausalSelfAttention(nn.Module):
    def __init__(self, config, layer_idx):
        super().__init__()
        self.n_head = config.n_head
        self.n_kv_head = config.n_kv_head
        self.n_embd = config.n_embd
        self.head_dim = self.n_embd // self.n_head
        assert self.n_embd % self.n_head == 0
        assert self.n_kv_head <= self.n_head and self.n_head % self.n_kv_head == 0
        self.c_q = nn.Linear(self.n_embd, self.n_head * self.head_dim, bias=False)
        self.c_k = nn.Linear(self.n_embd, self.n_kv_head * self.head_dim, bias=False)
        self.c_v = nn.Linear(self.n_embd, self.n_kv_head * self.head_dim, bias=False)
        self.c_proj = nn.Linear(self.n_embd, self.n_embd, bias=False)
        self.ve_gate_channels = 32
        self.ve_gate = nn.Linear(self.ve_gate_channels, self.n_kv_head, bias=False) if has_ve(layer_idx, config.n_layer) else None
```

This class implements the causal self-attention mechanism. Causal means each token can only attend to previous tokens (including itself), not future ones. This is essential for language modeling where we predict the next token.

The attention module has linear layers that project the input to queries (Q), keys (K), and values (V). The query and key projections go to `n_head * head_dim` dimensions, while the value projection only goes to `n_kv_head * head_dim`. This difference enables grouped query attention where multiple query heads share the same key-value heads, reducing computation. The `head_dim` is derived from embedding dimension divided by number of heads. The assertions ensure the dimensions divide evenly.

`c_proj` is an output projection that combines the attention outputs back to the model dimension. All these linear layers are bias-free (bias=False), a common practice in modern transformers.

The `ve_gate` implements the Value Embedding gate if this layer has value embeddings. It is a linear layer that takes the first 32 channels of the input and produces a gating scalar for each key-value head. If this layer does not have value embeddings, the gate is None.

```python
    def forward(self, x, ve, cos_sin, window_size):
        B, T, C = x.size()
        q = self.c_q(x).view(B, T, self.n_head, self.head_dim)
        k = self.c_k(x).view(B, T, self.n_kv_head, self.head_dim)
        v = self.c_v(x).view(B, T, self.n_kv_head, self.head_dim)

        # Value residual (ResFormer): mix in value embedding with input-dependent gate per head
        if ve is not None:
            ve = ve.view(B, T, self.n_kv_head, self.head_dim)
            gate = 2 * torch.sigmoid(self.ve_gate(x[..., :self.ve_gate_channels]))
            v = v + gate.unsqueeze(-1) * ve

        cos, sin = cos_sin
        q, k = apply_rotary_emb(q, cos, sin), apply_rotary_emb(k, cos, sin)
        q, k = norm(q), norm(k)

        y = fa3.flash_attn_func(q, k, v, causal=True, window_size=window_size)
        y = y.contiguous().view(B, T, -1)
        y = self.c_proj(y)
        return y
```

The forward method is where the computation happens. Input `x` has shape (B, T, C) for batch size, sequence length, and channel dimension. We compute Q, K, V by applying the linear projections and reshaping to separate the head dimension.

Value residual (ResFormer) adds value embeddings if present. The `ve` argument is the value embedding lookup: it is a tensor of shape (B, T, n_kv_head * head_dim) that gets reshaped to (B, T, n_kv_head, head_dim). The gate is computed as `2 * sigmoid(ve_gate(...))`. Why 2? Because sigmoid outputs values between 0 and 1, multiplying by 2 gives a range of 0 to 2, with 1 as the neutral point. The gate is applied per head and added to the values.

Then we apply rotary embeddings to Q and K using the precomputed cosine and sine tensors. After rotary, we apply RMS normalization to both Q and K. This is the QK normalization mentioned in the author's notes.

Then we call the Flash Attention function. The arguments are: `q`, `k`, `v` tensors, `causal=True` (restricts attention to previous tokens), and `window_size` which controls the sliding window pattern. If window_size is negative, it means full attention; if positive, it limits attention to that window. The result `y` has shape (B, T, n_head * head_dim). We call `.contiguous()` to ensure memory is contiguous before reshaping, then project through `c_proj`.

```python
class MLP(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.c_fc = nn.Linear(config.n_embd, 4 * config.n_embd, bias=False)
        self.c_proj = nn.Linear(4 * config.n_embd, config.n_embd, bias=False)

    def forward(self, x):
        x = self.c_fc(x)
        x = F.relu(x).square()
        x = self.c_proj(x)
        return x
```

The MLP (Multi-Layer Perceptron) is the feedforward network in each transformer block. It expands the embedding dimension by a factor of 4 (a common ratio), applies ReLU activation, then squares the result (ReLU squared), and projects back to the original dimension. The squaring is an unusual but effective activation function that produces non-negative outputs with a different shape than regular ReLU.

```python
class Block(nn.Module):
    def __init__(self, config, layer_idx):
        super().__init__()
        self.attn = CausalSelfAttention(config, layer_idx)
        self.mlp = MLP(config)

    def forward(self, x, ve, cos_sin, window_size):
        x = x + self.attn(norm(x), ve, cos_sin, window_size)
        x = x + self.mlp(norm(x))
        return x
```

A Block combines attention and MLP with residual connections. Note the normalization: both attention and MLP operate on normalized inputs, but the residual connection adds the original unnormalized input. This is different from the standard transformer where normalization comes after the residual. This "pre-norm" structure has become common because it stabilizes training for deep networks.

Now the full GPT model:

```python
class GPT(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.config = config
        self.window_sizes = self._compute_window_sizes(config)
        self.transformer = nn.ModuleDict({
            "wte": nn.Embedding(config.vocab_size, config.n_embd),
            "h": nn.ModuleList([Block(config, i) for i in range(config.n_layer)]),
        })
        self.lm_head = nn.Linear(config.n_embd, config.vocab_size, bias=False)
        self.resid_lambdas = nn.Parameter(torch.ones(config.n_layer))
        self.x0_lambdas = nn.Parameter(torch.zeros(config.n_layer))
        # Value embeddings
        head_dim = config.n_embd // config.n_head
        kv_dim = config.n_kv_head * head_dim
        self.value_embeds = nn.ModuleDict({
            str(i): nn.Embedding(config.vocab_size, kv_dim)
            for i in range(config.n_layer) if has_ve(i, config.n_layer)
        })
        # Rotary embeddings
        self.rotary_seq_len = config.sequence_len * 10
        cos, sin = self._precompute_rotary_embeddings(self.rotary_seq_len, head_dim)
        self.register_buffer("cos", cos, persistent=False)
        self.register_buffer("sin", sin, persistent=False)
```

The GPT class ties everything together. ModuleDict and ModuleList are PyTorch containers that properly register submodules so parameters are found during training.

`wte` is the token embedding table that maps token IDs to vectors. `h` is the list of transformer blocks. `lm_head` is the output layer that predicts the next token; it is tied to the input embeddings in some models, but here it is separate.

`resid_lambdas` and `x0_lambdas` are learnable scalar parameters that control the residual pathways. In the forward pass, `x = self.resid_lambdas[i] * x + self.x0_lambdas[i] * x0`. These allow the model to learn how much of the original input to preserve at each layer, a form of adaptive residual scaling.

`value_embeds` is a dictionary of embedding tables for layers that have value embeddings. The embedding dimension is `kv_dim` (key-value dimension), not the full `n_embd`.

Rotary embeddings are precomputed for a much longer sequence length (`rotary_seq_len = sequence_len * 10`) than we actually use, allowing the model to handle longer contexts than it was trained on by leveraging the rotary extrapolation property. These are registered as buffers, they are part of the model state but not trainable parameters.

```python
    @torch.no_grad()
    def init_weights(self):
        # Embedding and unembedding
        torch.nn.init.normal_(self.transformer.wte.weight, mean=0.0, std=1.0)
        torch.nn.init.normal_(self.lm_head.weight, mean=0.0, std=0.001)
```

Weight initialization is critical for training stability. This method initializes all parameters carefully. Token embeddings are initialized with normal distribution mean 0 std 1. The output head (`lm_head`) gets a much smaller std of 0.001.

```python
        # Transformer blocks
        n_embd = self.config.n_embd
        s = 3**0.5 * n_embd**-0.5
        for block in self.transformer.h:
            torch.nn.init.uniform_(block.attn.c_q.weight, -s, s)
            torch.nn.init.uniform_(block.attn.c_k.weight, -s, s)
            torch.nn.init.uniform_(block.attn.c_v.weight, -s, s)
            torch.nn.init.zeros_(block.attn.c_proj.weight)
            torch.nn.init.uniform_(block.mlp.c_fc.weight, -s, s)
            torch.nn.init.zeros_(block.mlp.c_proj.weight)
```

For the attention and MLP linear layers, the initializations follow specific schemes. `s = 3**0.5 * n_embd**-0.5` computes a uniform distribution bound. This comes from the LeCun initialization for ReLU activations, adapted for the square activation. The first linear layer in each branch (c_q, c_k, c_v, mlp.c_fc) get uniform weights in `[-s, s]`. The second projection layers (c_proj in both attention and MLP) are initialized to zero. This creates an identity-like behavior at initialization: initially, the residual branches just pass through the input without transforming it, which helps with gradient flow in deep networks.

```python
        # Per-layer scalars
        self.resid_lambdas.fill_(1.0)
        self.x0_lambdas.fill_(0.1)
```

The residual scaling parameters start with `resid_lambdas` at 1.0 (full residual) and `x0_lambdas` at 0.1 (small skip connection from the original input). This provides a stable initialization for the adaptive residual scheme.

```python
        # Value embeddings
        for ve in self.value_embeds.values():
            torch.nn.init.uniform_(ve.weight, -s, s)
        # Gate weights init to zero (sigmoid(0)=0.5, scaled by 2 -> 1.0 = neutral)
        for block in self.transformer.h:
            if block.attn.ve_gate is not None:
                torch.nn.init.zeros_(block.attn.ve_gate.weight)
```

Value embeddings get the same uniform initialization. The gate weights are initialized to zero because the gate computes `2 * sigmoid(ve_gate(...))`. When weights are zero, sigmoid(0) = 0.5, times 2 equals 1.0, which is neutral, the value embedding does not alter the original values initially. This allows the model to gradually learn whether to use value embeddings.

```python
        # Rotary embeddings
        head_dim = self.config.n_embd // self.config.n_head
        cos, sin = self._precompute_rotary_embeddings(self.rotary_seq_len, head_dim)
        self.cos, self.sin = cos, sin
        # Cast embeddings to bf16
        self.transformer.wte.to(dtype=torch.bfloat16)
        for ve in self.value_embeds.values():
            ve.to(dtype=torch.bfloat16)
```

The rotary embeddings are precomputed. Then we convert the token embedding and value embedding weights to bfloat16 precision. This is a memory optimization: bfloat16 (Brain Floating Point 16) uses half the memory of float32 while maintaining good numerical range. The model will run in mixed precision during training.

```python
    def _precompute_rotary_embeddings(self, seq_len, head_dim, base=10000, device=None):
        if device is None:
            device = self.transformer.wte.weight.device
        channel_range = torch.arange(0, head_dim, 2, dtype=torch.float32, device=device)
        inv_freq = 1.0 / (base ** (channel_range / head_dim))
        t = torch.arange(seq_len, dtype=torch.float32, device=device)
        freqs = torch.outer(t, inv_freq)
        cos, sin = freqs.cos(), freqs.sin()
        cos, sin = cos.bfloat16(), sin.bfloat16()
        cos, sin = cos[None, :, None, :], sin[None, :, None, :]
        return cos, sin
```

This computes the Rotary Position Embedding frequencies. The formula uses an exponential base (default 10000) and computes inverse frequencies for channel pairs. For each position, we compute cosine and sine values. The dimensions are arranged for broadcasting during the apply_rotary_emb operation. The result has shapes (1, seq_len, 1, head_dim/2) which broadcast across batch and head dimensions.

```python
    def _compute_window_sizes(self, config):
        pattern = config.window_pattern.upper()
        assert all(c in "SL" for c in pattern)
        long_window = config.sequence_len
        short_window = long_window // 2
        char_to_window = {"L": (long_window, 0), "S": (short_window, 0)}
        window_sizes = []
        for layer_idx in range(config.n_layer):
            char = pattern[layer_idx % len(pattern)]
            window_sizes.append(char_to_window[char])
        window_sizes[-1] = (long_window, 0)
        return window_sizes
```

This computes the sliding window attention pattern. The pattern string like "SSSL" means first three layers use short window (half sequence length), last layer uses full sequence. The window return value is a tuple (window_size, something) where the second element might be used for additional parameters but is zero here. The last layer is forced to full attention regardless of pattern, ensuring the model can always attend to the full context at the final layer.

```python
    def setup_optimizer(self, unembedding_lr=0.004, embedding_lr=0.2, matrix_lr=0.02,
                        weight_decay=0.0, adam_betas=(0.8, 0.95), scalar_lr=0.5):
        model_dim = self.config.n_embd
        matrix_params = list(self.transformer.h.parameters())
        value_embeds_params = list(self.value_embeds.parameters())
        embedding_params = list(self.transformer.wte.parameters())
        lm_head_params = list(self.lm_head.parameters())
        resid_params = [self.resid_lambdas]
        x0_params = [self.x0_lambdas]
```

This method creates the optimizer with carefully separated parameter groups, each with different learning rates. Different parts of the model benefit from different learning rates. The method collects parameters from different components: `matrix_params` are all the weight matrices in the transformer blocks, `value_embeds_params` are the value embedding tables, `embedding_params` are the token embeddings, `lm_head_params` are the output layer weights, and `resid_params` and `x0_params` are the scalar parameters.

```python
        assert len(list(self.parameters())) == (len(matrix_params) + len(embedding_params) +
            len(lm_head_params) + len(value_embeds_params) + len(resid_params) + len(x0_params))
```

This assertion verifies that we have accounted for all parameters in the model, nothing is missing or double-counted.

```python
        dmodel_lr_scale = (model_dim / 768) ** -0.5
        print(f"Scaling AdamW LRs by 1/sqrt({model_dim}/768) = {dmodel_lr_scale:.6f}")
```

This computes a scaling factor that adjusts learning rates based on model dimension. The formula says: if your model is larger than 768 dimensions, scale learning rates down proportionally to the square root of the ratio. This accounts for the fact that larger models need smaller learning rates for stability. The base of 768 is the default model dimension.

```python
        param_groups = [
            dict(kind='adamw', params=lm_head_params, lr=unembedding_lr * dmodel_lr_scale, betas=adam_betas, eps=1e-10, weight_decay=0.0),
            dict(kind='adamw', params=embedding_params, lr=embedding_lr * dmodel_lr_scale, betas=adam_betas, eps=1e-10, weight_decay=0.0),
            dict(kind='adamw', params=value_embeds_params, lr=embedding_lr * dmodel_lr_scale, betas=adam_betas, eps=1e-10, weight_decay=0.0),
            dict(kind='adamw', params=resid_params, lr=scalar_lr * 0.01, betas=adam_betas, eps=1e-10, weight_decay=0.0),
            dict(kind='adamw', params=x0_params, lr=scalar_lr, betas=(0.96, 0.95), eps=1e-10, weight_decay=0.0),
        ]
```

We create parameter groups for the AdamW optimizer. Each group specifies which parameters it contains, the learning rate, betas (momentum coefficients), epsilon for numerical stability, and weight decay. Notice the different learning rates: unembedding (lm_head) gets 0.004, embeddings get 0.2, value embeddings same as embeddings, residual lambdas get 0.5% of scalar_lr (0.005), x0 lambdas get full scalar_lr (0.5), and importantly x0 uses different betas (0.96, 0.95). Weight decay is zero for all these groups; only the Muon optimizer group gets weight decay.

```python
        for shape in sorted({p.shape for p in matrix_params}):
            group_params = [p for p in matrix_params if p.shape == shape]
            param_groups.append(dict(
                kind='muon', params=group_params, lr=matrix_lr,
                momentum=0.95, ns_steps=5, beta2=0.95, weight_decay=weight_decay,
            ))
```

The matrix parameters are partitioned by shape. Each unique shape gets its own Muon parameter group. Muon is an optimizer designed specifically for large matrix parameters. It uses different hyperparameters: momentum 0.95, beta2 0.95, weight_decay as specified, and ns_steps=5 which controls orthogonalization iterations. The grouping by shape is important because Muon operates by stacking parameters of the same shape together and applying its special update rule.

```python
        optimizer = MuonAdamW(param_groups)
        for group in optimizer.param_groups:
            group["initial_lr"] = group["lr"]
        return optimizer
```

We create the combined MuonAdamW optimizer (which we will examine in detail later). We store the initial learning rates for each group because the training loop applies learning rate schedules that multiply these base rates.

Now the forward method:

```python
    def forward(self, idx, targets=None, reduction='mean'):
        B, T = idx.size()
        assert T <= self.cos.size(1)
        cos_sin = self.cos[:, :T], self.sin[:, :T]

        x = self.transformer.wte(idx)
        x = norm(x)
        x0 = x
        for i, block in enumerate(self.transformer.h):
            x = self.resid_lambdas[i] * x + self.x0_lambdas[i] * x0
            ve = self.value_embeds[str(i)](idx) if str(i) in self.value_embeds else None
            x = block(x, ve, cos_sin, self.window_sizes[i])
        x = norm(x)

        softcap = 15
        logits = self.lm_head(x)
        logits = logits.float()
        logits = softcap * torch.tanh(logits / softcap)

        if targets is not None:
            loss = F.cross_entropy(logits.view(-1, logits.size(-1)), targets.view(-1),
                                   ignore_index=-1, reduction=reduction)
            return loss
        return logits
```

The forward pass begins: `idx` is token IDs with shape (B, T). We extract the rotary embeddings for this sequence length (T). Then token embedding lookup `wte(idx)` gives us (B, T, n_embd). We apply RMS normalization immediately, that is the prenorm setup.

We save `x0` as the initial normalized embeddings for use in the residual scaling formula.

Then we loop through each transformer block. Before each block, we apply the adaptive residual: `self.resid_lambdas[i] * x + self.x0_lambdas[i] * x0`. This mixes the current activations with the original input. Then we look up the value embedding for this layer if it exists, passing the token IDs `idx`. The value embedding is shared across positions, it is an embedding table that maps tokens to value vectors, similar to but separate from token embeddings. Then we pass through the block.

After all blocks, we normalize the final activations. Then we compute the lm_head (output projection) to get logits over the vocabulary. The logits are cast to float32 to avoid numerical issues. Then we apply softcapping: `softcap * tanh(logits / softcap)`. This limits the magnitude of logits to ±15, which stabilizes training and prevents extreme values that could cause numerical issues. The value 15 is fixed in this code; it is not a hyperparameter the agent can change (though it could if it modified the code).

If targets are provided (during training), we compute cross-entropy loss. The `ignore_index=-1` means any target value of -1 is masked out. The loss is averaged or summed according to the `reduction` argument. Without targets (during inference), we return the logits.

The MuonAdamW optimizer is a sophisticated combined optimizer that uses Muon for matrix parameters and AdamW for everything else. It is over 130 lines of code. I'll explain it thoroughly but more compactly.

```python
polar_express_coeffs = [
    (8.156554524902461, -22.48329292557795, 15.878769915207462),
    (4.042929935166739, -2.808917465908714, 0.5000178451051316),
    (3.8916678022926607, -2.772484153217685, 0.5060648178503393),
    (3.285753657755655, -2.3681294933425376, 0.46449024233003106),
    (2.3465413258596377, -1.7097828382687081, 0.42323551169305323),
]
```

These are hardcoded coefficients used in the "polar express" orthogonalization procedure. This comes from research on optimizing large matrices. Each tuple (a, b, c) is used in an iteration to orthogonalize gradients. The first iteration uses the first tuple, second uses second, etc. `ns_steps` in the config controls how many iterations (up to 5) we perform.

```python
@torch.compile(dynamic=False, fullgraph=True)
def adamw_step_fused(p, grad, exp_avg, exp_avg_sq, step_t, lr_t, beta1_t, beta2_t, eps_t, wd_t):
    p.mul_(1 - lr_t * wd_t)
    exp_avg.lerp_(grad, 1 - beta1_t)
    exp_avg_sq.lerp_(grad.square(), 1 - beta2_t)
    bias1 = 1 - beta1_t ** step_t
    bias2 = 1 - beta2_t ** step_t
    denom = (exp_avg_sq / bias2).sqrt() + eps_t
    step_size = lr_t / bias1
    p.add_(exp_avg / denom, alpha=-step_size)
```

This is the AdamW update step, written in a custom fused form suitable for `torch.compile`. Let's translate: First, apply weight decay: `p = p * (1 - lr * wd)`. Then update the exponential moving averages: `exp_avg = exp_avg * beta1 + grad * (1-beta1)`, and similarly for `exp_avg_sq` using `beta2`. Bias correction divides by `(1 - beta^step)` to account for initialization at zero. The step size is lr divided by bias-corrected first moment. Finally we update the parameter: `p = p - step_size * exp_avg / (sqrt(exp_avg_sq) + eps)`.

The arguments are tensors because this function is compiled and needs to avoid recompilation when hyperparameters change; using 0-D CPU tensors that get filled with current values allows the compiled code to be reused.

```python
@torch.compile(dynamic=False, fullgraph=True)
def muon_step_fused(stacked_grads, stacked_params, momentum_buffer, second_momentum_buffer,
                    muon_momentum_t, lr_t, wd_t, beta2_t, ns_steps, red_dim):
    # Nesterov momentum
    momentum = muon_momentum_t.to(stacked_grads.dtype)
    momentum_buffer.lerp_(stacked_grads, 1 - momentum)
    g = stacked_grads.lerp_(momentum_buffer, momentum)
```

Muon starts with Nesterov momentum: it updates the momentum buffer like `momentum = momentum * momentum + grad * (1-momentum)`. Then it computes a Nesterov lookahead gradient `g = grad + momentum * momentum_buffer`. This is different from regular momentum and provides better convergence.

```python
    # Polar express orthogonalization
    X = g.bfloat16()
    X = X / (X.norm(dim=(-2, -1), keepdim=True) * 1.02 + 1e-6)
    if g.size(-2) > g.size(-1):
        for a, b, c in polar_express_coeffs[:ns_steps]:
            A = X.mT @ X
            B = b * A + c * (A @ A)
            X = a * X + X @ B
    else:
        for a, b, c in polar_express_coeffs[:ns_steps]:
            A = X @ X.mT
            B = b * A + c * (A @ A)
            X = a * X + B @ X
    g = X
```

This is the most exotic part. The gradient matrix is normalized to unit norm per row or column (depending on shape). Then we apply a series of iterations that perform a matrix-valued Newton step towards orthogonality. The coefficients (a,b,c) come from approximating the matrix sign function or polar decomposition. The goal is to make the update directions more orthogonal, which improves optimization for large matrices. This is what makes Muon special compared to regular Adam.

```python
    # NorMuon variance reduction
    beta2 = beta2_t.to(g.dtype)
    v_mean = g.float().square().mean(dim=red_dim, keepdim=True)
    red_dim_size = g.size(red_dim)
    v_norm_sq = v_mean.sum(dim=(-2, -1), keepdim=True) * red_dim_size
    v_norm = v_norm_sq.sqrt()
    second_momentum_buffer.lerp_(v_mean.to(dtype=second_momentum_buffer.dtype), 1 - beta2)
    step_size = second_momentum_buffer.clamp_min(1e-10).rsqrt()
    scaled_sq_sum = (v_mean * red_dim_size) * step_size.float().square()
    v_norm_new = scaled_sq_sum.sum(dim=(-2, -1), keepdim=True).sqrt()
    final_scale = step_size * (v_norm / v_norm_new.clamp_min(1e-10))
    g = g * final_scale.to(g.dtype)
```

NorMuon is a variance reduction technique for large matrices. It computes the per-row (or per-column) variance of the gradient and normalizes the step size to keep the noise level constant. The mathematics here are sophisticated but the effect is to stabilize training for very large matrices by scaling updates according to the second moment statistics in a normalized way.

```python
    # Cautious weight decay + parameter update
    lr = lr_t.to(g.dtype)
    wd = wd_t.to(g.dtype)
    mask = (g * stacked_params) >= 0
    stacked_params.sub_(lr * g + lr * wd * stacked_params * mask)
```

The final step applies a "cautious" weight decay: weight decay is only applied when the gradient and parameter have the same sign (both positive or both negative). If they have opposite signs, the weight decay term is zeroed. This prevents updates that might push parameters in a harmful direction. The parameter update subtracts the learning rate times the processed gradient, plus weight decay if applicable.

```python
class MuonAdamW(torch.optim.Optimizer):
    """Combined optimizer: Muon for 2D matrix params, AdamW for others."""

    def __init__(self, param_groups):
        super().__init__(param_groups, defaults={})
        # 0-D CPU tensors to avoid torch.compile recompilation when values change
        self._adamw_step_t = torch.tensor(0.0, dtype=torch.float32, device="cpu")
        # ... similar tensors for other hyperparameters
```

The MuonAdamW optimizer is a custom subclass of torch.optim.Optimizer. Its `__init__` creates special 0-D CPU tensors to hold running hyperparameter values. These are used as arguments to the compiled step functions because compiled functions need tensors with fixed types; using these CPU tensors that we fill with current values allows the compiled code to be reused instead of recompiled every time a learning rate or beta changes.

```python
    def _step_adamw(self, group):
        for p in group['params']:
            if p.grad is None:
                continue
            grad = p.grad
            state = self.state[p]
            if not state:
                state['step'] = 0
                state['exp_avg'] = torch.zeros_like(p)
                state['exp_avg_sq'] = torch.zeros_like(p)
            state['step'] += 1
            self._adamw_step_t.fill_(state['step'])
            self._adamw_lr_t.fill_(group['lr'])
            self._adamw_beta1_t.fill_(group['betas'][0])
            self._adamw_beta2_t.fill_(group['betas'][1])
            self._adamw_eps_t.fill_(group['eps'])
            self._adamw_wd_t.fill_(group['weight_decay'])
            adamw_step_fused(p, grad, state['exp_avg'], state['exp_avg_sq'],
                            self._adamw_step_t, self._adamw_lr_t, self._adamw_beta1_t,
                            self._adamw_beta2_t, self._adamw_eps_t, self._adamw_wd_t)
```

For AdamW groups, we iterate over parameters, initialize state if not present (step counter, exponential moving averages), update the step counter, fill the CPU tensors with current hyperparameter values, and call the compiled `adamw_step_fused` function.

```python
    def _step_muon(self, group):
        params = group['params']
        if not params:
            return
        p = params[0]
        state = self.state[p]
        num_params = len(params)
        shape, device, dtype = p.shape, p.device, p.dtype
        if "momentum_buffer" not in state:
            state["momentum_buffer"] = torch.zeros(num_params, *shape, dtype=dtype, device=device)
        if "second_momentum_buffer" not in state:
            state_shape = (num_params, shape[-2], 1) if shape[-2] >= shape[-1] else (num_params, 1, shape[-1])
            state["second_momentum_buffer"] = torch.zeros(state_shape, dtype=dtype, device=device)
```

For Muon, parameters are grouped by shape. We check that there is at least one parameter in the group. We get the state associated with the first parameter (all parameters in the group share state). The momentum buffer has shape `(num_params, ...)` to store separate momentum for each parameter in the group. The second_momentum_buffer has a reduced shape: it stores statistics per row (if matrix is wide) or per column (if matrix is tall), not per parameter element.

```python
        red_dim = -1 if shape[-2] >= shape[-1] else -2
        stacked_grads = torch.stack([p.grad for p in params])
        stacked_params = torch.stack(params)
        self._muon_momentum_t.fill_(group["momentum"])
        self._muon_beta2_t.fill_(group["beta2"] if group["beta2"] is not None else 0.0)
        self._muon_lr_t.fill_(group["lr"] * max(1.0, shape[-2] / shape[-1])**0.5)
        self._muon_wd_t.fill_(group["weight_decay"])
        muon_step_fused(stacked_grads, stacked_params,
                        state["momentum_buffer"], state["second_momentum_buffer"],
                        self._muon_momentum_t, self._muon_lr_t, self._muon_wd_t,
                        self._muon_beta2_t, group["ns_steps"], red_dim)
        torch._foreach_copy_(params, list(stacked_params.unbind(0)))
```

We determine the reduction dimension: if there are more rows than columns (tall matrix), we reduce over columns (dim -1), otherwise over rows (dim -2). We stack all gradients and parameters in the group into tensors with an extra leading dimension. We fill the CPU tensors with current hyperparameters. The learning rate is scaled by `sqrt(shape[-2] / shape[-1])` to normalize for aspect ratio. Then we call the compiled muon_step_fused which updates the stacked_params in-place. Finally, we copy the stacked parameters back to the individual parameter tensors using `torch._foreach_copy_`.

The step method simply iterates over parameter groups and dispatches to the appropriate step function:

```python
    @torch.no_grad()
    def step(self):
        for group in self.param_groups:
            if group['kind'] == 'adamw':
                self._step_adamw(group)
            elif group['kind'] == 'muon':
                self._step_muon(group)
```

All parameter updates happen with `torch.no_grad()` to avoid tracking these operations in the autograd graph.

Now we come to the hyperparameters section. This is a key place where the AI agent makes changes:

```python
# Model architecture
ASPECT_RATIO = 64       # model_dim = depth * ASPECT_RATIO
HEAD_DIM = 128          # target head dimension for attention
WINDOW_PATTERN = "SSSL" # sliding window pattern: L=full, S=half context

# Optimization
TOTAL_BATCH_SIZE = 2**19 # ~524K tokens per optimizer step
EMBEDDING_LR = 0.6      # learning rate for token embeddings (Adam)
UNEMBEDDING_LR = 0.004  # learning rate for lm_head (Adam)
MATRIX_LR = 0.04        # learning rate for matrix parameters (Muon)
SCALAR_LR = 0.5         # learning rate for per-layer scalars (Adam)
WEIGHT_DECAY = 0.2      # cautious weight decay for Muon
ADAM_BETAS = (0.8, 0.95) # Adam beta1, beta2
WARMUP_RATIO = 0.0      # fraction of time budget for LR warmup
WARMDOWN_RATIO = 0.5    # fraction of time budget for LR warmdown
FINAL_LR_FRAC = 0.0     # final LR as fraction of initial

# Model size
DEPTH = 8               # number of transformer layers
DEVICE_BATCH_SIZE = 128  # per-device batch size (reduce if OOM)
```

These are the knobs the agent can twiddle. ASPECT_RATIO controls the relationship between depth and width: the model dimension is computed as `depth * ASPECT_RATIO`, then rounded up to a multiple of HEAD_DIM. So with DEPTH=8 and ASPECT_RATIO=64, base_dim = 512, but then it gets rounded to 768 to be divisible by 128. The agent can change ASPECT_RATIO to control width independently of depth.

HEAD_DIM is the target size of each attention head. The actual head_dim is derived from n_embd / n_head, but this constant influences the calculation indirectly. The pattern determines which layers use sliding window attention.

TOTAL_BATCH_SIZE is the total number of tokens per optimizer step, achieved through gradient accumulation. The actual batch that fits on GPU is DEVICE_BATCH_SIZE * MAX_SEQ_LEN tokens. We accumulate gradients over `grad_accum_steps = TOTAL_BATCH_SIZE / (DEVICE_BATCH_SIZE * MAX_SEQ_LEN)` steps before doing an optimizer step. This allows us to have a large effective batch size even with limited GPU memory.

Learning rates are separated by parameter group: embedding and value embeddings get 0.6, lm_head gets 0.004, matrix parameters get 0.04, and scalar parameters get 0.5. These are quite different and tuned empirically. The agent might try adjusting them.

The betas for AdamW are (0.8, 0.95) which differ from the usual (0.9, 0.999). The author mentions the agent discovered the AdamW betas were wrong, suggesting this was one of the improvements. Indeed 0.8 and 0.95 are unusual but might work better for this specific setup.

WARMUP_RATIO and WARMDOWN_RATIO control learning rate scheduling. With 0 warmup and 0.5 warmdown, the learning rate stays constant for the first half of training then linearly decays to zero in the second half. FINAL_LR_FRAC=0 means it decays all the way to zero (linear warmdown to zero).

DEPTH is the number of transformer layers. This is one of the most significant architectural knobs; more layers means a deeper network with more capacity. The agent can change this. DEVICE_BATCH_SIZE controls how many sequences fit in GPU memory; if the agent makes the model bigger, it might need to reduce this to avoid out-of-memory errors.

Now the setup code:

```python
t_start = time.time()
torch.manual_seed(42)
torch.cuda.manual_seed(42)
torch.set_float32_matmul_precision("high")
device = torch.device("cuda")
autocast_ctx = torch.amp.autocast(device_type="cuda", dtype=torch.bfloat16)
H100_BF16_PEAK_FLOPS = 989.5e12
```

We set random seeds for reproducibility. We set float32 matmul precision to "high" which enables TensorFloat-32 (TF32) on Ampere+ GPUs for faster matrix multiplication. We set the device to CUDA. We create an autocast context manager for automatic mixed precision: operations will use bfloat16 where safe, improving speed and memory. We define the theoretical peak FLOPS of an H100 in bfloat16 for computing MFU (Model FLOPs Utilization) later.

```python
tokenizer = Tokenizer.from_directory()
vocab_size = tokenizer.get_vocab_size()
print(f"Vocab size: {vocab_size:,}")
```

We load the tokenizer from the cache. It will use the actual vocab size from training, which might differ from the default 32768 in GPTConfig.

```python
def build_model_config(depth):
    base_dim = depth * ASPECT_RATIO
    model_dim = ((base_dim + HEAD_DIM - 1) // HEAD_DIM) * HEAD_DIM
    num_heads = model_dim // HEAD_DIM
    return GPTConfig(
        sequence_len=MAX_SEQ_LEN, vocab_size=vocab_size,
        n_layer=depth, n_head=num_heads, n_kv_head=num_heads, n_embd=model_dim,
        window_pattern=WINDOW_PATTERN,
    )
```

This function constructs the GPTConfig given a depth. The model dimension is computed as `depth * ASPECT_RATIO` then rounded up to be divisible by HEAD_DIM. This ensures head_dim divides n_embd evenly. The number of heads is simply n_embd / HEAD_DIM. This auto-scaling means when the agent changes DEPTH, the width changes proportionally (via ASPECT_RATIO) to maintain a roughly constant ratio.

```python
config = build_model_config(DEPTH)
print(f"Model config: {asdict(config)}")
```

We build the configuration using the current DEPTH constant and print all the settings. This helps the agent see what configuration was used in each experiment.

```python
with torch.device("meta"):
    model = GPT(config)
model.to_empty(device=device)
model.init_weights()
```

This is a memory-efficient initialization technique. We first create the model on the "meta" device, which is a special PyTorch device that allocates no memory for tensors; it just tracks shapes and dtypes. Then we call `to_empty(device=device)` which allocates real memory on the GPU but does not copy any data, the tensors are still uninitialized. Then we call `init_weights()` to fill all parameters with proper initial values directly on the GPU. This avoids ever having two copies of the model in memory during initialization.

```python
param_counts = model.num_scaling_params()
print("Parameter counts:")
for key, value in param_counts.items():
    print(f"  {key:24s}: {value:,}")
num_params = param_counts['total']
num_flops_per_token = model.estimate_flops()
print(f"Estimated FLOPs per token: {num_flops_per_token:e}")
```

We compute parameter counts broken down by component, and total them. We also estimate FLOPs per token, which is important for computing Model FLOPs Utilization (MFU) later. The estimate accounts for attention FLOPs which depend on window sizes.

```python
tokens_per_fwdbwd = DEVICE_BATCH_SIZE * MAX_SEQ_LEN
assert TOTAL_BATCH_SIZE % tokens_per_fwdbwd == 0
grad_accum_steps = TOTAL_BATCH_SIZE // tokens_per_fwdbwd
```

We compute how many forward-backward passes we need to accumulate before an optimizer step. TOTAL_BATCH_SIZE is 2^19 = 524288 tokens. DEVICE_BATCH_SIZE * MAX_SEQ_LEN = 128 * 2048 = 262144 tokens per forward pass. So grad_accum_steps = 524288 / 262144 = 2. That means we do 2 forward-backward passes, accumulating gradients, before stepping the optimizer. If the agent changes DEVICE_BATCH_SIZE, this assertion would catch an invalid configuration where TOTAL_BATCH_SIZE is not divisible by tokens_per_fwdbwd.

```python
optimizer = model.setup_optimizer(
    unembedding_lr=UNEMBEDDING_LR,
    embedding_lr=EMBEDDING_LR,
    scalar_lr=SCALAR_LR,
    adam_betas=ADAM_BETAS,
    matrix_lr=MATRIX_LR,
    weight_decay=WEIGHT_DECAY,
)
```

We create the MuonAdamW optimizer, passing all the hyperparameters from the top-level constants. The method will scale the learning rates by the model dimension scaling factor and create the parameter groups.

```python
model = torch.compile(model, dynamic=False)
```

We compile the model using PyTorch's `torch.compile` to generate optimized kernels. This can significantly speed up training (often 1.3-2x). `dynamic=False` says we do not need dynamic shapes, which simplifies compilation and can produce faster code. The compilation happens on the first forward pass.

```python
train_loader = make_dataloader(tokenizer, DEVICE_BATCH_SIZE, MAX_SEQ_LEN, "train")
x, y, epoch = next(train_loader)  # prefetch first batch
```

We create the training dataloader (which we studied in prepare.py) and immediately fetch the first batch to warm up the dataloader and avoid stalls later.

```python
print(f"Time budget: {TIME_BUDGET}s")
print(f"Gradient accumulation steps: {grad_accum_steps}")
```

Print some diagnostics.

Now the learning rate scheduling functions:

```python
def get_lr_multiplier(progress):
    if progress < WARMUP_RATIO:
        return progress / WARMUP_RATIO if WARMUP_RATIO > 0 else 1.0
    elif progress < 1.0 - WARMDOWN_RATIO:
        return 1.0
    else:
        cooldown = (1.0 - progress) / WARMDOWN_RATIO
        return cooldown * 1.0 + (1 - cooldown) * FINAL_LR_FRAC
```

Progress is a number between 0 and 1 representing how far we are through the time budget. The learning rate multiplier starts with linear warmup if WARMUP_RATIO > 0, then stays at 1.0 during the middle plateau, then linearly warmdown to FINAL_LR_FRAC during the last WARMDOWN_RATIO portion. With default WARMUP_RATIO=0, we skip warmup. WARMDOWN_RATIO=0.5 means for the last 150 seconds of the 300 second budget, we linearly decay the learning rate to zero (FINAL_LR_FRAC=0). This schedule helps the model converge to a final solution.

```python
def get_muon_momentum(step):
    frac = min(step / 300, 1)
    return (1 - frac) * 0.85 + frac * 0.95
```

Muon momentum starts at 0.85 and linearly increases to 0.95 over the first 300 steps. This is a schedule different from the main time budget, it is based on step count, not wall-clock time. This provides another knob the agent could modify.

```python
def get_weight_decay(progress):
    return WEIGHT_DECAY * (1 - progress)
```

Weight decay linearly decreases from WEIGHT_DECAY (0.2) to 0 over the course of training. This is a form of learning rate schedule for the weight decay parameter itself.

Now the training loop:

```python
t_start_training = time.time()
smooth_train_loss = 0
total_training_time = 0
step = 0

while True:
    torch.cuda.synchronize()
    t0 = time.time()
    for micro_step in range(grad_accum_steps):
        with autocast_ctx:
            loss = model(x, y)
        train_loss = loss.detach()
        loss = loss / grad_accum_steps
        loss.backward()
        x, y, epoch = next(train_loader)
```

We start timing. `smooth_train_loss` is an exponential moving average. `total_training_time` tracks how much actual training time has elapsed (excluding the first few warmup steps). `step` counts optimizer steps.

The outer `while True` loop runs until we break when time's up. Each iteration corresponds to one optimizer step (after grad accumulation). Inside, we synchronize CUDA to get accurate timing. We record start time t0. Then we do `grad_accum_steps` micro-steps. In each micro-step, we run the forward pass through the model to get loss (with autocast for mixed precision). We detach the loss to keep it for logging. We divide the loss by the number of accumulation steps so that when we call `loss.backward()`, the gradients are averaged appropriately across accumulation steps. Then we fetch the next batch. Note that `next(train_loader)` is a blocking call; the dataloader yields pre-prepared batches.

```python
    # Progress and schedules
    progress = min(total_training_time / TIME_BUDGET, 1.0)
    lrm = get_lr_multiplier(progress)
    muon_momentum = get_muon_momentum(step)
    muon_weight_decay = get_weight_decay(progress)
    for group in optimizer.param_groups:
        group["lr"] = group["initial_lr"] * lrm
        if group['kind'] == 'muon':
            group["momentum"] = muon_momentum
            group["weight_decay"] = muon_weight_decay
```

We compute the current progress fraction. We get the LR multiplier from the schedule. We compute Muon-specific hyperparameters. Then we update every parameter group's learning rate by multiplying the initial LR by the common multiplier. For Muon groups, we also update momentum and weight decay dynamically. This means AdamW groups use fixed betas and eps, but Muon adjusts two hyperparameters on the fly.

```python
    optimizer.step()
    model.zero_grad(set_to_none=True)
```

Perform the actual parameter update using the combined MuonAdamW optimizer. Then zero out gradients with `set_to_none=True` which is slightly faster than setting to zero because it allows memory to be reused.

```python
    train_loss_f = train_loss.item()
```

Extract the scalar loss value as a Python float for printing/logging.

```python
    # Fast fail: abort if loss is exploding or NaN
    if math.isnan(train_loss_f) or train_loss_f > 100:
        print("FAIL")
        exit(1)
```

A sanity check: if loss becomes NaN (not a number) or explodes above 100, we crash immediately. This prevents wasting time on obviously broken runs. The agent will see this as a crash and try something else.

```python
    torch.cuda.synchronize()
    t1 = time.time()
    dt = t1 - t0
```

We synchronize again to get a precise end time. dt is the elapsed wall-clock time for this optimizer step, including data loading, forward, backward, and optimizer update.

```python
    if step > 10:
        total_training_time += dt
```

We do not count the first 10 steps toward total_training_time. Why? The first few steps include compilation overhead (torch.compile needs to compile kernels on first run) and other startup costs that are not representative of steady-state training. We exclude them to make the 5-minute budget measure actual training throughput rather than setup costs.

```python
    # Logging
    ema_beta = 0.9
    smooth_train_loss = ema_beta * smooth_train_loss + (1 - ema_beta) * train_loss_f
    debiased_smooth_loss = smooth_train_loss / (1 - ema_beta**(step + 1))
```

We maintain an exponential moving average of the training loss with decay 0.9. To get a less biased estimate (since the EMA starts at 0), we divide by `(1 - ema_beta^(step+1))`. This gives a more accurate view of the current loss level.

```python
    pct_done = 100 * progress
    tok_per_sec = int(TOTAL_BATCH_SIZE / dt)
    mfu = 100 * num_flops_per_token * TOTAL_BATCH_SIZE / dt / H100_BF16_PEAK_FLOPS
    remaining = max(0, TIME_BUDGET - total_training_time)
```

We compute the percentage of time budget used. We compute tokens per second by dividing total tokens processed per optimizer step by dt. We compute Model FLOPs Utilization (MFU): the ratio of actual FLOPs achieved to the theoretical peak FLOPs of the GPU. This tells us how efficiently we are using the hardware. We compute remaining time.

```python
    print(f"\rstep {step:05d} ({pct_done:.1f}%) | loss: {debiased_smooth_loss:.6f} | lrm: {lrm:.2f} | dt: {dt*1000:.0f}ms | tok/sec: {tok_per_sec:,} | mfu: {mfu:.1f}% | epoch: {epoch} | remaining: {remaining:.0f}s    ", end="", flush=True)
```

We print a one-line status update using carriage return `\r` to overwrite the previous line, keeping the console clean. The format includes step number, percentage, smoothed loss, LR multiplier, step time in ms, throughput, MFU, epoch count, and remaining time.

```python
    # GC management (Python's GC causes ~500ms stalls)
    if step == 0:
        gc.collect()
        gc.freeze()
        gc.disable()
    elif (step + 1) % 5000 == 0:
        gc.collect()
    step += 1
```

Garbage collection can cause unpredictable pauses. On the first step, we collect all garbage, then freeze the objects and disable GC, which prevents further collections and avoids those stalls. Every 5000 steps we do a manual collection to avoid memory leaks, but hopefully this is infrequent. This is a performance optimization.

```python
    # Time's up - but only stop after warmup steps so we do not count compilation
    if step > 10 and total_training_time >= TIME_BUDGET:
        break
```

We break the loop when the total training time (excluding first 10 steps) reaches the time budget. The condition `step > 10` ensures we complete at least a couple of steps after excluding compilation overhead.

After the loop:

```python
print()  # newline after \r training log

total_tokens = step * TOTAL_BATCH_SIZE
```

We print a newline to move off the status line. We compute total tokens processed during training.

```python
# Final eval
model.eval()
with autocast_ctx:
    val_bpb = evaluate_bpb(model, tokenizer, DEVICE_BATCH_SIZE)
```

We switch the model to evaluation mode (affects dropout, batch norm if present). We run the evaluation function from prepare.py to compute the validation bits per byte. This is the key metric.

```python
# Final summary
t_end = time.time()
startup_time = t_start_training - t_start
steady_state_mfu = 100 * num_flops_per_token * TOTAL_BATCH_SIZE * (step - 10) / total_training_time / H100_BF16_PEAK_FLOPS if total_training_time > 0 else 0
peak_vram_mb = torch.cuda.max_memory_allocated() / 1024 / 1024

print("---")
print(f"val_bpb:          {val_bpb:.6f}")
print(f"training_seconds: {total_training_time:.1f}")
print(f"total_seconds:    {t_end - t_start:.1f}")
print(f"peak_vram_mb:     {peak_vram_mb:.1f}")
print(f"mfu_percent:      {steady_state_mfu:.2f}")
print(f"total_tokens_M:   {total_tokens / 1e6:.1f}")
print(f"num_steps:        {step}")
print(f"num_params_M:     {num_params / 1e6:.1f}")
print(f"depth:            {DEPTH}")
```

We compute statistics. `total_seconds` is wall clock time from start to end (including setup, compilation). `peak_vram_mb` shows how much GPU memory we used at peak. `steady_state_mfu` recomputes MFU using only the steps past the first 10, giving a cleaner measure of sustained utilization. We print all the key metrics in a format that the agent's program.md expects to parse.

That covers the core training script. The agent can modify any constant in the hyperparameters section, the model architecture within the GPT class, the optimizer settings, the training loop logic, or even add new components entirely. The only constraints are that the script must complete within the time budget (or it gets killed) and must produce the expected output format.

## Deep Dive: program.md - How You Tell the Agent What To Do

The `program.md` file is the instruction set for the autonomous agent. It is written in plain markdown and explains step by step what the agent should do. The AI agent (such as Claude or Codex) reads this file and follows its instructions to perform the autonomous research loop. Let me walk through its contents.

```markdown
# autoresearch

This is an experiment to have the LLM do its own research.

## Setup

To set up a new experiment, work with the user to:

1. **Agree on a run tag**: propose a tag based on today's date (e.g. `mar5`). The branch `autoresearch/<tag>` must not already exist ,  this is a fresh run.
2. **Create the branch**: `git checkout -b autoresearch/<tag>` from current master.
3. **Read the in-scope files**: The repo is small. Read these files for full context:
   - `README.md` ,  repository context.
   - `prepare.py` ,  fixed constants, data prep, tokenizer, dataloader, evaluation. Do not modify.
   - `train.py` ,  the file you modify. Model architecture, optimizer, training loop.
4. **Verify data exists**: Check that `~/.cache/autoresearch/` contains data shards and a tokenizer. If not, tell the human to run `uv run prepare.py`.
5. **Initialize results.tsv**: Create `results.tsv` with just the header row. The baseline will be recorded after the first run.
6. **Confirm and go**: Confirm setup looks good.

Once you get confirmation, kick off the experimentation.
```

The setup phase ensures everything is ready. The agent starts by proposing a run tag, a short name for the experiment series, typically based on the date (like "mar15" for March 15). This tag becomes part of a git branch name: `autoresearch/mar15`. Using git branches allows the agent to experiment freely without losing progress; successful changes advance the branch, failures reset back.

The instructions explicitly tell the agent to create a new branch from the current master. This ensures a clean starting point.

The agent is instructed to read all the important files to understand the context. This is crucial because the agent needs to know what the code does before it can modify it intelligently. The README gives overall context; prepare.py defines the fixed evaluation and data pipeline; train.py is the mutable training script.

Then the agent must verify that data and tokenizer exist in the cache. If not, the human needs to run prepare.py first. The agent should not run this itself because prepare.py involves downloading a huge dataset and training a tokenizer, these are one-time operations that the human should do.

Next, the agent creates a `results.tsv` file with a header row. TSV stands for tab-separated values. The columns are commit, val_bpb, memory_gb, status, description. The agent will fill this file with the outcomes of each experiment.

Finally, the agent confirms with the user that setup looks correct, then begins experimentation.

```markdown
## Experimentation

Each experiment runs on a single GPU. The training script runs for a **fixed time budget of 5 minutes** (wall clock training time, excluding startup/compilation). You launch it simply as: `uv run train.py`.

**What you CAN do:**
- Modify `train.py` ,  this is the only file you edit. Everything is fair game: model architecture, optimizer, hyperparameters, training loop, batch size, model size, etc.

**What you CANNOT do:**
- Modify `prepare.py`. It is read-only. It contains the fixed evaluation, data loading, tokenizer, and training constants (time budget, sequence length, etc).
- Install new packages or add dependencies. You can only use what is already in `pyproject.toml`.
- Modify the evaluation harness. The `evaluate_bpb` function in `prepare.py` is the ground truth metric.

**The goal is simple: get the lowest val_bpb.** Since the time budget is fixed, you do not need to worry about training time ,  it is always 5 minutes. Everything is fair game: change the architecture, the optimizer, the hyperparameters, the batch size, the model size. The only constraint is that the code runs without crashing and finishes within the time budget.

**VRAM** is a soft constraint. Some increase is acceptable for meaningful val_bpb gains, but it should not blow up dramatically.

**Simplicity criterion**: All else being equal, simpler is better. A small improvement that adds ugly complexity is not worth it. Conversely, removing something and getting equal or better results is a great outcome ,  that is a simplification win. When evaluating whether to keep a change, weigh the complexity cost against the improvement magnitude. A 0.001 val_bpb improvement that adds 20 lines of hacky code? Probably not worth it. A 0.001 val_bpb improvement from deleting code? Definitely keep. An improvement of ~0 but much simpler code? Keep.
```

The experimentation section defines the rules. The agent has complete freedom to modify train.py except it cannot touch prepare.py, cannot install new packages, and cannot change the evaluation metric. The objective is val_bpb, and since time is fixed, the agent is effectively optimizing for the best model achievable in exactly 5 minutes. This encourages the agent to find configurations that train quickly and effectively.

There is also a thoughtful simplicity criterion: improvements should be weighed against code complexity. This prevents the agent from accumulating hacky workarounds. A change that simplifies while maintaining performance is considered positive even without a metric gain.

```markdown
**The first run**: Your very first run should always be to establish the baseline, so you will run the training script as is.
```

Before making any changes, the agent should run the current train.py without modifications to establish a baseline val_bpb. All subsequent experiments compare against this baseline or the current best.

```markdown
## Output format

Once the script finishes it prints a summary like this:

```
---
val_bpb:          0.997900
training_seconds: 300.1
total_seconds:    325.9
peak_vram_mb:     45060.2
mfu_percent:      39.80
total_tokens_M:   499.6
num_steps:        953
num_params_M:     50.3
depth:           8
```

Note that the script is configured to always stop after 5 minutes, so depending on the computing platform of this computer the numbers might look different. You can extract the key metric from the log file:

```
grep "^val_bpb:" run.log
```
```

The agent needs to know what output to parse. The training script prints these lines at the end. The agent should capture them to record the results.

```markdown
## Logging results

When an experiment is done, log it to `results.tsv` (tab-separated, NOT comma-separated ,  commas break in descriptions).

The TSV has a header row and 5 columns:

```
commit	val_bpb	memory_gb	status	description
```

1. git commit hash (short, 7 chars)
2. val_bpb achieved (e.g. 1.234567) ,  use 0.000000 for crashes
3. peak memory in GB, round to .1f (e.g. 12.3 ,  divide peak_vram_mb by 1024) ,  use 0.0 for crashes
4. status: `keep`, `discard`, or `crash`
5. short text description of what this experiment tried

Example:

```
commit	val_bpb	memory_gb	status	description
a1b2c3d	0.997900	44.0	keep	baseline
b2c3d4e	0.993200	44.2	keep	increase LR to 0.04
c3d4e5f	1.005000	44.0	discard	switch to GeLU activation
d4e5f6g	0.000000	0.0	crash	double model width (OOM)
```
```

The agent must meticulously record every experiment in the TSV file. For each run, it should:
- Get the latest git commit hash (short version) after editing train.py
- Extract val_bpb from the output, or 0.0 if the run crashed
- Extract peak_vram_mb, convert to GB with one decimal
- Determine status: if val_bpb improved compared to the current best kept experiment, status is "keep"; if val_bpb is not better, status is "discard" (but still recorded to show the exploration); if the run crashed or NaN or OOM, status is "crash"
- Write a concise description of what changed

The TSV uses tabs, not commas, because descriptions might contain commas.

```markdown
## The experiment loop

The experiment runs on a dedicated branch (e.g. `autoresearch/mar5` or `autoresearch/mar5-gpu0`).

LOOP FOREVER:

1. Look at the git state: the current branch/commit we are on
2. Tune `train.py` with an experimental idea by directly hacking the code.
3. git commit
4. Run the experiment: `uv run train.py > run.log 2>&1` (redirect everything ,  do NOT use tee or let output flood your context)
5. Read out the results: `grep "^val_bpb:\|^peak_vram_mb:" run.log`
6. If the grep output is empty, the run crashed. Run `tail -n 50 run.log` to read the Python stack trace and attempt a fix. If you cannot get things to work after more than a few attempts, give up.
7. Record the results in the tsv (NOTE: do not commit the results.tsv file, leave it untracked by git)
8. If val_bpb improved (lower), you "advance" the branch, keeping the git commit
9. If val_bpb is equal or worse, you git reset back to where you started

The idea is that you are a completely autonomous researcher trying things out. If they work, keep. If they do not, discard. And you are advancing the branch so that you can iterate. If you feel like you are getting stuck in some way, you can rewind but you should probably do this very very sparingly (if ever).

**Timeout**: Each experiment should take ~5 minutes total (+ a few seconds for startup and eval overhead). If a run exceeds 10 minutes, kill it and treat it as a failure (discard and revert).

**Crashes**: If a run crashes (OOM, or a bug, or etc.), use your judgment: If it is something dumb and easy to fix (e.g. a typo, a missing import), fix it and re-run. If the idea itself is fundamentally broken, just skip it, log "crash" as the status in the tsv, and move on.

**NEVER STOP**: Once the experiment loop has begun (after the initial setup), do NOT pause to ask the human if you should continue. Do NOT ask "should I keep going?" or "is this a good stopping point?". The human might be asleep, or gone from a computer and expects you to continue working *indefinitely* until you are manually stopped. You are autonomous. If you run out of ideas, think harder ,  read papers referenced in the code, re-read the in-scope files for new angles, try combining previous near-misses, try more radical architectural changes. The loop runs until the human interrupts you, period.

As an example use case, a user might leave you running while they sleep. If each experiment takes you ~5 minutes then you can run approx 12/hour, for a total of about 100 over the duration of the average human sleep. The user then wakes up to experimental results, all completed by you while they slept!
```

This section is the heart of the autonomous loop. It gives the agent explicit instructions on how to conduct research.

The loop begins with inspecting the current git state to know where we are. The agent then comes up with an experimental idea, this is where the AI's creativity comes in. It directly edits train.py to implement its idea. Then it commits the change with a message that will later appear in the results.

The agent runs `uv run train.py > run.log 2>&1` which redirects both stdout and stderr to a log file. Using `>` rather than `tee` is important because otherwise the agent's context would be flooded with output, making it harder to reason.

After the run finishes (or crashes), the agent extracts val_bpb and peak_vram_mb using grep, which searches for lines starting with those labels. If the greps return nothing, the script likely crashed before reaching the summary; the agent should read the last 50 lines of the log to see the error.

The agent records the results in the TSV file, but crucially it should NOT commit the TSV file. The TSV is a log of all experiments including discarded ones; it remains untracked by git so that it does not clutter the repository history.

Then the agent decides: if the val_bpb is lower than the current best, the change is kept. The branch stays at this new commit, making it the new baseline for future iterations. If the val_bpb is not better, the agent resets the branch back to the previous commit (`git reset --hard HEAD~1` or similar), discarding the change. This creates a hill-climbing process where only improvements survive.

Timeouts: if an experiment exceeds 10 minutes, the agent should kill it and treat it as a failure. Possibly the agent made the model too big or batch size too large, causing extremely slow training.

Crashes: if an experiment crashes, the agent should try to fix trivial mistakes like typos, but if the crash is due to fundamental issues (e.g., out-of-memory because the model is too big for GPU), it should skip and log as crash.

Finally, and most importantly: the agent should never stop. It should run indefinitely until the human interrupts. This autonomy is what enables overnight experiments.

## The GPT Model Explained

Now I will explain the GPT model architecture in simple, non-technical terms. The model is a transformer, which is the same type of neural network that powers GPT-3, GPT-4, and many other modern language models. But instead of describing it in academic terms, let me use an analogy.

Imagine you are reading a long book, and your task is to predict the next word on each page. You do not start from scratch on every page; you carry context from what you have already read. A transformer model does this by maintaining a "memory" of all previous words in the sequence. The key component is the attention mechanism, which allows each position to look back at all previous positions and decide what information to gather.

In GPT, the model processes text one token at a time. Tokens are pieces of words, often roughly corresponding to syllables or common subwords. The model first converts each token into a vector, a list of numbers that represents its meaning and role in the sequence. This is the token embedding.

Then the model passes this vector through a series of layers, called transformer blocks. Each block has two main components: an attention layer and a feedforward neural network. The attention layer lets each position "read" information from all previous positions, while the feedforward layer independently processes each position's representation.

The model in this project uses several advanced techniques:

Value Residuals (ResFormer): normally in attention, we compute query, key, and value from the same input. Here, the authors add extra value embeddings that come from the token IDs directly. This is like having a second source of information about each token's meaning that is separate from the main stream. A learned gate determines how much of this value residual to add.

QK Normalization: the queries and keys are normalized before computing attention scores. This stabilizes training by keeping the dot products in a reasonable range. Without normalization, the attention scores can become too large or too small, making training unstable.

Sliding Window Attention: instead of allowing each token to attend to all previous tokens (which would be expensive for long sequences), some layers use a limited window. The pattern "SSSL" means the first three layers can only look back half the sequence length, while the final layer can look at the full context. This reduces computation dramatically and introduces a bias toward more recent information in early layers, while still allowing the final layer to see everything.

Rotary Position Embeddings: these are a clever way to encode position information without adding separate positional vectors. Instead, the token vectors are rotated by an amount that depends on their position. This rotation preserves the vector's norm and has nice mathematical properties that make it easy for the model to learn relative positions.

RMS Normalization: the model uses Root Mean Square normalization instead of LayerNorm. It is simpler: you just divide by the root mean square of the vector's elements. No learned shift and scale parameters. This speeds up computation and reduces memory.

Per-layer scalars: the model has learnable scalars `resid_lambdas` and `x0_lambdas` that control how much of the residual paths contribute. At each block, the computation is `x = lambda_res * x + lambda_x0 * x0 + block(x)`. This gives the model fine-grained control over information flow.

Softcapping: the final logits are passed through a tanh multiplied by 15, limiting them to ±15. This prevents extreme values that could cause numerical instability in the cross-entropy loss.

The model also uses grouped query attention: the number of key-value heads (`n_kv_head`) is equal to `n_head` by default, but the architecture supports using fewer KV heads, which saves computation without much quality loss.

All these design choices are subject to change by the agent. The agent might discover that different activation functions work better, or that different normalization schemes improve results, or that adjusting the window pattern yields gains. The author's notes mention specific discoveries: QKnorm lacked a scaler multiplier (so attention was too diffuse), Value Embeddings needed regularization, banded attention was too conservative, AdamW betas were wrong, weight decay schedule needed tuning, network initialization needed adjustment.

In other words, the manual tuning that the author had already done was still imperfect. The autonomous agent, by systematically exploring the space of configurations, found tangible improvements that stacked to produce an 11% better model in terms of "Time to GPT-2" leaderboard metric.

## The Optimizer: MuonAdamW

Optimizers are algorithms that adjust the parameters of a neural network to minimize the loss function. The most famous is Stochastic Gradient Descent (SGD), which simply subtracts the gradient times a learning rate. But modern optimizers like Adam and AdamW add momentum and adaptive learning rates per parameter.

MuonAdamW is a hybrid optimizer that uses Muon for matrix parameters and AdamW for everything else. Let me explain why this is interesting and how it works.

AdamW is the standard optimizer for most deep learning tasks. It maintains two moving averages of gradients for each parameter: the first moment (mean) and the second moment (uncentered variance). The update rule scales the learning rate by the square root of the second moment (so parameters with large gradients get smaller effective steps) and uses the first moment as the direction. The "W" stands for decoupled weight decay, meaning weight decay is applied separately from the gradient-based update, which has been shown to work better than L2 regularization in Adam.

Muon is a recent optimizer (from 2024) that is specifically designed for large matrix parameters, such as the weight matrices in transformer layers. It is based on the idea that for large matrices, the optimization landscape can be improved by orthogonalizing the gradient directions or using Newton-like corrections on the matrix manifold.

The Muon algorithm does several things:

1. Nesterov Momentum: similar to SGD with momentum but with a lookahead.
2. Polar Express Orthogonalization: This step attempts to make the gradient directions more orthogonal. The mathematics involve matrix iterations using the coefficients we saw. The goal is to approximate the polar decomposition or matrix sign function, which helps prevent the optimization from getting stuck in ill-conditioned regions.
3. NorMuon Variance Reduction: This normalizes the gradient based on its statistics to keep the noise level consistent across different parameter shapes and scales. It sums the per-row second moments and rescales the gradient to maintain constant variance, which stabilizes training.

The key insight is that these steps are particularly beneficial for 2D weight matrices (like linear layer weights) but might be overkill or even harmful for scalar parameters (like bias terms or the lambdas). That is why the MuonAdamW optimizer partitions parameters: matrix parameters that are 2D get Muon, everything else gets AdamW. In this code, matrix parameters are those in the transformer blocks (the `matrix_params` list). The scalar lambdas, embeddings, and LM head use AdamW.

Also note that Muon applies cautious weight decay: weight decay is only applied when the gradient and parameter have the same sign. This prevents the optimizer from pushing a parameter away from zero when the gradient already indicates movement in the opposite direction. This is a form of sign-aware regularization that protects against harmful weight decay.

The MuonAdamW class handles both types of updates in one training step. It uses `torch.compile` to fuse operations for speed. The implementation is quite complex due to the need to handle variable shapes efficiently and avoid recompilation. The use of 0-D CPU tensors for hyperparameters is a trick to keep the compiled graph from needing to be recompiled every time hyperparameters change, if these were regular tensors on GPU with different values each step, the compiler would think it is a different graph each time.

For the AI agent, the important hyperparameters related to the optimizer are:
- MATRIX_LR: learning rate for Muon parameters (default 0.04)
- EMBEDDING_LR: LR for token and value embeddings (0.6)
- UNEMBEDDING_LR: LR for the output head (0.004)
- SCALAR_LR: LR for lambdas (0.5)
- WEIGHT_DECAY: weight decay strength for Muon (0.2)
- ADAM_BETAS: betas for AdamW groups (0.8, 0.95)
- (And the Muon-specific momentum default 0.95, ns_steps 5, beta2 0.95)

The agent might discover that different values yield better val_bpb within 5 minutes. For example, the author's notes mention that the agent found AdamW betas were messed up, suggesting maybe 0.8 and 0.95 are not optimal.

## The Training Loop: Where the Magic Happens

The training loop is the engine that drives the entire learning process. I've already described it in detail in the train.py walkthrough, but let me step back and explain the big picture.

The goal of training a language model is to adjust its millions of parameters so that it becomes better at predicting what comes next in text. We do this by showing it lots of examples and minimizing the cross-entropy loss, which measures how surprised the model is by the actual next token.

The training loop does the following repeatedly until the time budget expires:

1. Pick a batch of training data: This comes from the dataloader, which streams tokens from the dataset. The batch size in tokens is TOTAL_BATCH_SIZE, achieved via gradient accumulation. Each batch consists of multiple sequences, each of length 2048 tokens.

2. Forward pass: The model takes the input tokens and produces logits (probability distributions over the vocabulary). We compute the loss by comparing these logits to the true next tokens.

3. Backward pass: The gradients of the loss with respect to all parameters are computed via backpropagation. This tells us which direction to move each parameter to reduce the loss.

4. Optimizer step: The MuonAdamW optimizer uses the gradients to update all parameters according to their respective rules. For AdamW parameters, it uses the AdamW formula; for matrix parameters, it uses the sophisticated Muon algorithm involving orthogonalization and variance normalization.

5. Schedules: As training progresses, we adjust learning rates and Muon-specific hyperparameters. The learning rate goes through a plateau then linear warmdown to zero. Muon momentum ramps up from 0.85 to 0.95 over first 300 steps. Weight decay decays linearly.

6. Logging: We print a status line showing the current loss, throughput, MFU, and estimated remaining time. We also maintain an exponential moving average of the loss to smooth out noise.

7. Cleanup: We manage garbage collection to avoid pauses.

The loop breaks when the accumulated training time (excluding first 10 steps) reaches 300 seconds. Then we run a final evaluation on the validation set to compute val_bpb. This is the single number that summarizes the model's quality after 5 minutes of training.

Why exactly 5 minutes? This fixed budget creates a fair competition between different model configurations. If some changes make training slower (e.g., larger model, more layers, slower attention), they still have to perform well within the same time constraint. This encourages the agent to find configurations that are not just accurate but also efficient. The time budget is wall-clock time measured after startup, so compilation and initial setup do not count. This ensures each experiment gives roughly comparable results.

## The Autonomous Experiment Loop

Now let me bring it all together: step-by-step what the agent does, day after day, while the human sleeps.

### Step 1: Initial Setup

The human opens a terminal, navigates to the autoresearch directory, and runs `uv run prepare.py` if they have not already. This downloads a small portion (by default 10 shards) of the climbmix-400b dataset and trains the BPE tokenizer. This might take a few minutes.

Then the human opens their AI assistant (Claude, Codex, or similar) and gives it an initial prompt in the repository. Something like: "Hi, have a look at program.md and let's kick off a new experiment. Let's do the setup first."

The AI agent reads program.md and follows the setup instructions:
- It checks what date it is and proposes a branch name like `autoresearch/mar15`.
- It checks that this branch does not already exist; if it does, it might suggest `autoresearch/mar15-2` or similar.
- It creates the branch from master: `git checkout -b autoresearch/mar15`.
- It reads README.md, prepare.py, and train.py to understand the codebase.
- It verifies that `~/.cache/autoresearch/` contains data and tokenizer. If not, it tells the human they need to run prepare.py.
- It creates an empty `results.tsv` with just the header row: `commit	val_bpb	memory_gb	status	description`
- It confirms with the human: "Setup complete. I'll start with a baseline run. Is that okay?"

The human says yes. The autonomous mode begins.

### Step 2: Baseline Run

The agent runs `uv run train.py > run.log 2>&1`. This is the baseline configuration with all default hyperparameters. After about 5 minutes (plus some compilation overhead), the run finishes. The agent extracts:
- `val_bpb: 0.997900`
- `peak_vram_mb: 45060.2`
- other metrics

It logs this to results.tsv:
```
a1b2c3d	0.997900	44.0	keep	baseline
```
using the actual short commit hash. It does not commit results.tsv. The branch now stays at this commit.

### Step 3: First Experiment

The agent thinks. What should it try first? It looks at train.py and considers possible changes. Maybe it decides to increase the embedding learning rate from 0.6 to 0.8. It edits the line `EMBEDDING_LR = 0.6` to `EMBEDDING_LR = 0.8`. It commits this change with message "increase embedding LR to 0.8".

Then it runs `uv run train.py > run.log 2>&1` again. After 5 minutes, it gets a val_bpb of 0.995500 (improvement) and peak VRAM 44.1 GB. It logs:
```
b2c3d4e	0.995500	44.1	keep	increase embedding LR to 0.8
```
Because val_bpb improved (lower), it keeps the change. The branch head remains at this new commit. This becomes the new baseline for future comparisons.

### Step 4: Iterate

The agent continues. Now it might try decreasing the warmdown ratio from 0.5 to 0.4. Or it might increase DEPTH from 8 to 10, making the model deeper (but also wider automatically via ASPECT_RATIO). Or it might change WINDOW_PATTERN from "SSSL" to "LLL" to give all layers full attention. Or it might tweak the Muon momentum from default computed schedule to a fixed higher value. Or it might add L2 regularization to value embeddings. Or it might adjust the ADAM_BETAS. Or it might change the softcap value. Or it might add dropout. The possibilities are vast but constrained to train.py.

Each change is committed, experimentally evaluated for 5 minutes, and compared to the current best. If better, it stays; if not, it is thrown away by resetting the branch. The agent might sometimes combine multiple changes from previous successful experiments, or try variations on a theme.

The agent also monitors memory. If a change causes VRAM to blow up dramatically (say from 44 GB to 80 GB), it might consider that too costly even if val_bpb slightly improves, according to the simplicity criterion (which implicitly penalizes high memory as it reduces accessibility). But the instruction says VRAM is a soft constraint, not a hard limit; some increase is acceptable for meaningful gains.

As the agent experiments, it builds a history in results.tsv. Some experiments are kept, some discarded, some crash. The kept ones represent a sequence of real improvements. After about 100 experiments over 2 days, the agent may have discovered 20 improvements that all stack, meaning the final model is the baseline plus all those changes applied cumulatively.

The final model could have, for example, DEPTH=10, ASPECT_RATIO adjusted, different learning rates, different window pattern, maybe additional normalization, different initializations, maybe using regularization on value embeddings, and so on. Each change individually improved val_bpb; together they compound.

The human wakes up and finds:
- The git repository at branch `autoresearch/mar15` with a linear history of 20 commits (each one a successful improvement).
- A `results.tsv` file with 100 rows (some kept, many discarded, some crashes).
- A `progress.png` image generated by analysis.ipynb showing the val_bpb frontier declining over experiments.

The human can then take these discovered improvements and apply them to a larger model (e.g., nanochat with depth=24), where they indeed stack and produce an 11% reduction in "Time to GPT-2" on the leaderboard.

## What Did the Agent Actually Discover?

According to the author's notes, the agent found about 20 changes that improved validation loss. All of these were additive and transferred to larger models. Here are some of the notable discoveries mentioned:

- "It noticed an oversight that my parameterless QKnorm did not have a scaler multiplier attached, so my attention was too diffuse. The agent found multipliers to sharpen it, pointing to future work."

In the code, we saw that Q and K are normalized with RMSNorm: `q, k = norm(q), norm(k)`. This is parameterless normalization. The agent discovered that adding a learned scalar multiplier to the normalized Q and K (so they are multiplied by a trainable coefficient before computing attention) would sharpen the attention distribution and improve results. This is interesting because the original author had not thought to add such a scaler.

- "It found that the Value Embeddings really like regularization and I was not applying any (oops)."

Value Embeddings are additional token-specific embeddings added to the attention values. The agent discovered that adding weight decay or L2 regularization to these embeddings prevented overfitting and improved validation loss. The initial code had no weight decay applied to value embeddings (they were in an AdamW group with weight_decay=0). The agent might have moved them to the Muon group with weight_decay>0 or added explicit regularization.

- "It found that my banded attention was too conservative (i forgot to tune it)."

Sliding window attention uses the window sizes determined by WINDOW_PATTERN. The agent likely tuned the pattern, the short window size, or maybe even added dynamic adjustments. The initial pattern "SSSL" might have been too restrictive; perhaps "SSSS" or "SSLL" or adjusting SHORT_WINDOW from half to some other fraction helped.

- "It found that AdamW betas were all messed up."

AdamW uses two beta parameters: beta1 for momentum and beta2 for second moment. The defaults were (0.8, 0.95). The agent discovered that different values, perhaps (0.9, 0.95) or (0.85, 0.99) or something else, worked better. Beta1 too low means less momentum smoothing; beta2 too low means the variance estimate adapts more quickly. The right values can stabilize training.

- "It tuned the weight decay schedule."

The weight decay for Muon is scheduled as `WEIGHT_DECAY * (1 - progress)`, meaning it linearly decays to zero. The agent might have discovered a different schedule (e.g., keep constant, or increase, or use a cosine decay) or adjusted the initial WEIGHT_DECAY value.

- "It tuned the network initialization."

The weight initialization scheme in `init_weights` uses specific scales for different layers. The agent might have found that adjusting the `s` value or using a different distribution improved training stability or final loss.

Beyond these specific items, the agent likely tuned learning rates across all groups, adjusted the ASPECT_RATIO to change width vs depth trade-offs, experimented with different activation functions (maybe swapped ReLU^2 for something else), added or removed normalization layers, changed the torch.compile settings, or tweaked gradient accumulation steps.

The fact that all 20 changes were additive means that each one independently improved val_bpb when applied on top of the current best. This suggests the search found orthogonal improvements that complemented each other.

The final result: an 11% improvement in "Time to GPT-2". This is a benchmark metric that measures how long it takes to train a model to GPT-2 level performance. The baseline was 2.02 hours; after applying all the agent's discoveries, it dropped to 1.80 hours. That is a 13-minute improvement for reaching the same quality, which is significant in research and development cost.

## Why This Matters: The Future of AI Research

Autoresearch demonstrates a shift in how AI systems can be developed. Traditionally, human researchers spend countless hours designing architectures, tuning hyperparameters, and running experiments. This work is slow, expensive, and often relies on intuition and experience.

Imagine a future where AI research itself becomes largely automated. You could define the research problem, provide a codebase and compute budget, and let swarms of AI agents explore the space of possible improvements. Each agent might specialize in a particular aspect: one focuses on architecture, another on optimizers, another on data preprocessing. They could communicate their findings, share successful configurations, and collaborate on deeper insights.

The autoresearch project is a minimal proof-of-concept. It shows that with a constrained problem (single GPU, 5-minute budget, one modifiable file), a single language model agent can indeed make meaningful progress overnight. The discoveries it made were real improvements that the original human researcher had missed despite years of experience. This suggests that automated search can augment human intuition and uncover optimizations that might otherwise remain undiscovered.

For smaller research teams or individual developers, this democratizes capability. You do not need a large lab with many grad students trying different ideas. You can have one or two AI agents running experiments on your laptop or a single GPU, making progress while you sleep or focus on higher-level design.

For large AI labs, this points toward fully autonomous research pipelines. The frontier labs already use massive compute to train large models. Automated experiment management can multiply their productivity. They could run thousands of small-scale experiments in parallel, each exploring a different niche of the configuration space, and promote the most promising findings to larger scales. The author hints at this: "All LLM frontier labs will do this. It is the final boss battle."

The autoresearch project is a tangible demonstration that AI-driven research is not science fiction. It is possible today with existing models, existing hardware, and relatively simple code. The fact that a single file of 630 lines can be the target of autonomous optimization shows how accessible this frontier is.

## Is This Safe? Risk and Safety Considerations

When we hear about AI that can improve itself, many people feel uneasy. Questions naturally arise: Is this safe? Could it go rogue? Will it take all the jobs? These concerns deserve serious thought. Autonomous AI research is still in its very early days. The systems being built today are far from the independent, sentient AI of science fiction. They are specialized tools that assist human researchers, not replace them. The code written by these systems must still be checked, tested, and supervised by experts. Think of it like a very fast assistant who drafts reports but needs a manager to review the work. That human oversight remains essential.

The AI community is aware of potential risks. Many organizations have safety teams dedicated to understanding how advanced AI could be misused or cause harm. They develop guidelines and best practices for responsible research. Autonomous coding agents are designed with limitations. They operate within narrow boundaries, improving a specific piece of code rather than making open-ended decisions. This containment reduces the chance of unintended consequences.

As for jobs, history suggests that automation often creates new types of work while eliminating some old ones. AI researchers themselves are in high demand. Tools that make their work easier could actually increase the need for human insight. But we should not dismiss worries about displacement. The transition matters, and society needs policies to help people adapt. The key is to recognize that this research is a tool. Its ultimate impact depends on how we choose to use it. The path forward requires both enthusiasm for the benefits and careful attention to the risks. That balanced approach is what responsible innovation looks like.

## Conclusion

Andrej Karpathy's autoresearch project is a minimalist autonomous research system. It consists of three files: `prepare.py` (fixed infrastructure for data and tokenization), `train.py` (the 630-line training script that the AI agent modifies), and `program.md` (instructions for the agent). The agent runs an endless loop: modify train.py, run a 5-minute experiment, evaluate val_bpb, keep or discard the change based on whether it improved. Over roughly 100 experiments in 2 days, the agent discovered about 20 improvements that stacked to produce an 11% better model. Key discoveries included adding a scaler to QK normalization, regularizing value embeddings, tuning banded attention, correcting AdamW betas, adjusting weight decay schedule, and improving weight initialization.

The project showcases how language models can act as autonomous researchers, iterating on code and empirical results without human intervention. The design is clever: fixed time budget ensures fair comparisons, single modifiable file keeps scope manageable, and results logging tracks progress. The AI agent uses its reasoning abilities to propose changes, interpret outcomes, and plan next steps. This is a glimpse into a future where AI research becomes automated, accelerating the pace of discovery and changing the role of human scientists from manual experimenters to directors of autonomous research swarms. The code is open source and designed to be run by anyone with a GPU, inviting the community to experiment with their own agents and prompts to see what improvements they can discover.
