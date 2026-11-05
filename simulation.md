
import matplotlib.pyplot as plt
import numpy as np

# --- 1. PARAMETERS ---
n_hands = 10000
n_simulations = 500
win_rate_per_hand = 0.315
std_dev_hand = 9.0 #empirical benchmark
starting_bankroll = 2000

# --- 2. SIMULATION ---
hands = np.random.normal(
    loc=win_rate_per_hand, scale=std_dev_hand, size=(n_simulations, n_hands)
)
paths = np.cumsum(hands, axis=1) + starting_bankroll

# --- 3. RISK OF RUIN & STATS ---
min_balances = np.min(paths, axis=1)
risk_of_ruin = (np.sum(min_balances <= 0) / n_simulations) * 100

final_balances = paths[:, -1]
pct_losing = (
    np.sum(final_balances < starting_bankroll) / n_simulations
) * 100

print(f"Risk of Ruin: {risk_of_ruin:.2f}%")
print(f"Percentage of careers ending down at 10k hands: {pct_losing:.2f}%")

# --- 4. CALCULATE PERCENTILES ---
p5 = np.percentile(paths, 5, axis=0)
p50 = np.percentile(paths, 50, axis=0)
p95 = np.percentile(paths, 95, axis=0)

# Grab the final bankroll value at hand #10,000 for the 5th and 95th percentiles
final_p5 = p5[-1]
final_p95 = p95[-1]
final_p50 = p50[-1]

print(f"--- FINAL BANKROLL SPREAD AT HAND 10,000 ---")
print(f"Pessimistic (5th Pctl): {final_p5:.1f} BB")
print(f"Median (50th Pctl):     {final_p50:.1f} BB")
print(f"Optimistic (95th Pctl): {final_p95:.1f} BB")
# --- 5. PLOT THE CHART ---
plt.figure(figsize=(10, 5))

# Faded individual paths
plt.plot(paths.T, color="dodgerblue", alpha=0.02)

# Percentile and reference lines
plt.plot(p50, color="darkblue", linewidth=2, label="Median Path (50th Pctl)")
plt.plot(
    p95,
    color="forestgreen",
    linestyle="--",
    linewidth=1.5,
    label="95th Percentile (Optimistic)",)
plt.plot(
    p5,
    color="darkorange",
    linestyle="--",
    linewidth=1.5,
    label="5th Percentile (Pessimistic)",)

# Reference lines
plt.axhline(
    starting_bankroll, color="green", linestyle=":", label="Starting Bankroll")
plt.axhline(
    0, color="crimson", linestyle="--", linewidth=2, label="Absorbing Barrier (Ruin)")

plt.title("Monte Carlo Bankroll Trajectories & Percentile Bands")
plt.xlabel("Hands Played")
plt.ylabel("Bankroll (Big Blinds)")
plt.legend(loc="upper left")
plt.grid(True, alpha=0.3)
plt.show()
