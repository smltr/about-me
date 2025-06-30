# LanguageLabs Deep Dive

## Project Overview

  **Context:**
  - Self-funded AI startup exploring agentic systems (2023-2024, ~1.5 yrs)
  - Thoughtful research and modelling combined with prototyping
  - Transitioned back to employment after strategic decision to join a team

  **What I wanted to end up with:**
  - Agentic systems that can think before responding
  - Meta-learning systems for large language models with tool use
    - mainly file editing and interfacing with terminal
  - Self-modifying AI agents with persistent memory
  - Novel approaches to modeling intelligence, learning, and memory
  - Defining intelligence

  **What got me interested:**
  - LLMs seemed to have the intelligence for autonomous work but lacked the right framework to iterate on tasks over time
  - I viewed my internal moment-to-moment Q&A as something an LLM is capable of doing

## Core Research Areas

  Some things I worked on either by prototyping or whiteboarding/notebook sketches

  **1. Prompt Loop Agents**
  - **Problem:** LLMs can't iterate on tasks over time
  - **Approach:**
    - Designed "prompt loop instructions" for LLMs to iterate on tasks
    - Built systems for LLMs to check their own progress
    - Tool interface through specific language tokens
    - Allowed the system to edit its own base prompt loop instruction set
    - Set up system to periodically ask for input from me, adding a human in the loop
  - **Result:**
    - Never achieved stability. Systems would devolve into loops or incoherent states
    - I formed interesting perspectives on the problems of self-referential systems, and intelligence ultimately requiring external input

  **2. Intentional Neural Network Construction**
  - **Problem:** Current neural nets use random tweaks + performance judging
  - **Theory:**
    - Inference and learning as a combined mechanism
    - Systematic node strengthening based on incoming information
    - Inspired by Hebbian learning principles ("neurons that fire together, wire together")
    - Meta-learning where learning ability improves through acquired knowledge
    - I envisioned a system where as text is read, the placing of nodes and strenghtening of connections is systematically placed due to rules inspired by natural processes (namely hebbian learning)

  **3. Smart Nodes Architecture**
  - **Concept:** Abstract neural networks into clusters representing 'smart nodes', or language based functions
  - **Vision:**
    - Nodes as documents with instructions that LLMs can modify
    - Nodes have input and output, they send an error message back when inputs don't look as they should
      - this can be intelligently judged by the core LLM
    - Network of interconnected "experiences" and "skills"

## Key Insights & Hunches

  - Intelligence and memory are very intertwined

  - Human intelligence is developed both within an individual mind, but also throughout generations. Copying is key

  - Self-modification is weird.
    - Sleep?

  - Human memory as automatic tagging/retrieval
    - Put it where you will stumble upon it when you need it

  - The power of the right question

  - Common threads between neuroscience, psychology and ANNs

## Deciding to step back

  **Why I Stepped Back:**
  - Money ran out
  - Recognized value of joining established team vs. solo research
  - I had the passion, belief & focus, but lacked the experience and resources needed

  **What I Gained:**
  - Deep understanding of LLM capabilities and limitations, and an intuition of how to use them effectively while learning/thinking/working
  - Expertise in prompt engineering and context
  - Unique perspective on human-AI collaboration
  - A new sense of humility
  - Realized how important physical health is to my ability to focus and problem solve
  - An understanding of trying -> failing -> getting back up

Next, [approach.md](approach.md)
