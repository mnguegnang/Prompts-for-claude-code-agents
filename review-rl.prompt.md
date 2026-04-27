# Reinforcement Learning Review Framework

Domain-specific review checklist for RL codebases. Loaded by the Code Review agent when the project involves RL agents, environments, reward design, or policy optimization.

## 1. Environment & MDP Design

- Check for state space observability issues (e.g., missing historical context needed for decision making).
- Inspect reward design for "reward hacking" vulnerabilities, sparse reward bottlenecks, or unscaled rewards causing gradient explosion.
- Ensure environment resets (`env.reset()`) correctly clear all episodic state variables.
- Verify that observation/action spaces are correctly defined and match what the policy expects.

## 2. Algorithm & Training Loop Correctness

- Verify off-policy vs. on-policy alignment (e.g., preventing the improper use of a replay buffer with an on-policy algorithm like PPO).
- Check discounted return calculation (proper use of gamma, handling of terminal states vs. time-limit truncations).
- Inspect exploration vs. exploitation mechanisms (epsilon decay logic, entropy regularization).
- Verify advantage estimation (GAE lambda) and value function target computation.

## 3. Reproducibility & Evaluation

- Ensure random seeds are fixed across the agent, the environment, and action spaces.
- Verify that evaluation runs are strictly deterministic (e.g., `action = argmax(policy)` instead of sampling), unless stochasticity is explicitly required.
- Check for proper logging of episode returns, lengths, and success metrics across training.
