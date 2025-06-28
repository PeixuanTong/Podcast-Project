# Podcast-Project

import numpy as np
import pandas as pd

# Step 1: Setup
players = [f"P{i+1}" for i in range(12)]
payoffs = {
    "champion": 100,
    "finalist": 75,
    "semifinalist": 50,
    "quarterfinalist": 25,
    "first_round": 0
}

# Create a full 16-player bracket (pad with unseeded players if needed)
full_players = players + [f"U{i}" for i in range(16 - len(players))]

# Define pairwise win probabilities (simplified here)
# Assume ELO-derived or all 50/50 for now
win_probs = pd.DataFrame(0.5, index=full_players, columns=full_players)
np.fill_diagonal(win_probs.values, 0.0)

# Step 2: Simulate tournament
def simulate_tournament(players, win_probs):
    current_round = players[:]
    results = {p: 0 for p in players}

    # Round of 16
    next_round = []
    for i in range(0, len(current_round), 2):
        p1, p2 = current_round[i], current_round[i+1]
        win = np.random.choice([p1, p2], p=[win_probs.loc[p1, p2], win_probs.loc[p2, p1]])
        next_round.append(win)
        loser = p2 if win == p1 else p1
        results[loser] = payoffs["first_round"]

    # Quarterfinals
    current_round = next_round
    next_round = []
    for i in range(0, len(current_round), 2):
        p1, p2 = current_round[i], current_round[i+1]
        win = np.random.choice([p1, p2], p=[win_probs.loc[p1, p2], win_probs.loc[p2, p1]])
        next_round.append(win)
        loser = p2 if win == p1 else p1
        results[loser] = payoffs["quarterfinalist"]

    # Semifinals
    current_round = next_round
    next_round = []
    for i in range(0, len(current_round), 2):
        p1, p2 = current_round[i], current_round[i+1]
        win = np.random.choice([p1, p2], p=[win_probs.loc[p1, p2], win_probs.loc[p2, p1]])
        next_round.append(win)
        loser = p2 if win == p1 else p1
        results[loser] = payoffs["semifinalist"]

    # Final
    p1, p2 = next_round
    win = np.random.choice([p1, p2], p=[win_probs.loc[p1, p2], win_probs.loc[p2, p1]])
    results[win] = payoffs["champion"]
    loser = p2 if win == p1 else p1
    results[loser] = payoffs["finalist"]

    return results

# Step 3: Monte Carlo Simulation to estimate price
def estimate_expected_values(players, win_probs, n_sim=10000):
    total_scores = {p: 0 for p in players}
    for _ in range(n_sim):
        res = simulate_tournament(full_players, win_probs)
        for p in players:
            total_scores[p] += res.get(p, 0)
    return {p: total_scores[p] / n_sim for p in players}

# Step 4: Run and get fair price
expected_prices = estimate_expected_values(players, win_probs)
prices_df = pd.DataFrame.from_dict(expected_prices, orient='index', columns=["Fair Price"]).sort_values("Fair Price", ascending=False)
print(prices_df)
