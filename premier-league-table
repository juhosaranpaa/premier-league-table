# install if needed

import pandas as pd
from tabulate import tabulate  # Using tabulate for better formatting
import zipfile



# Download without unzipping
!kaggle datasets download -d davidcariboo/player-scores

# Unzip only the required files

with zipfile.ZipFile("player-scores.zip", 'r') as z:
    z.extract("clubs.csv")
    z.extract("games.csv")

print("Extracted clubs.csv and games.csv!")

# Load the data
games_df = pd.read_csv("games.csv", delimiter=",")  # Adjust delimiter if needed
clubs_df = pd.read_csv("clubs.csv", delimiter=",")  # Adjust delimiter if needed

# Convert "date" column to datetime format
games_df["date"] = pd.to_datetime(games_df["date"], errors="coerce")

# Find the latest match date in the dataset for Premier League (GB1)
latest_game_date = games_df[games_df["competition_id"] == "GB1"]["date"].max()

# Filter for Premier League (GB1) and Season 2024
filtered_games = games_df[(games_df["season"] == 2024) & (games_df["competition_id"] == "GB1")].copy()

# Merge to get club names instead of IDs
clubs_map = clubs_df.set_index("club_id")["name"].to_dict()
filtered_games["home_club_name"] = filtered_games["home_club_id"].map(clubs_map)
filtered_games["away_club_name"] = filtered_games["away_club_id"].map(clubs_map)

# Initialize points table and games played table
points_table = {}
games_played = {}

# Calculate points and games played
for _, row in filtered_games.iterrows():
    home_team = row["home_club_name"]
    away_team = row["away_club_name"]
    home_goals = row["home_club_goals"]
    away_goals = row["away_club_goals"]

    # Initialize team data if not already present
    points_table.setdefault(home_team, 0)
    points_table.setdefault(away_team, 0)
    games_played.setdefault(home_team, 0)
    games_played.setdefault(away_team, 0)

    # Count games played
    games_played[home_team] += 1
    games_played[away_team] += 1

    # Assign points based on match results
    if home_goals > away_goals:
        points_table[home_team] += 3  # Home team wins
    elif home_goals < away_goals:
        points_table[away_team] += 3  # Away team wins
    else:
        points_table[home_team] += 1  # Draw
        points_table[away_team] += 1  # Draw

# Function to clean team names
def clean_team_name(name):
    return (
        name.replace(" Football Club", "")
        .replace("Association ", "")
        .replace(" and Hove Albion", "")
        .replace(" Wanderers", "")
        .strip()
    )

# Convert dictionary to a DataFrame
points_df = pd.DataFrame({
    "Team": [clean_team_name(team) for team in points_table.keys()],
    "Points": points_table.values(),
    "GP": [games_played[team] for team in points_table.keys()]
})

# Sort by points in descending order and reset index to create "Position"
points_df = points_df.sort_values(by="Points", ascending=False).reset_index(drop=True)
points_df.index += 1  # Start index from 1 instead of 0
points_df.index.name = "Position"  # Rename index to "Position"

# Save results
points_df.to_csv("premier_league_points_2024.csv", index=True)  # Save with "Position" as index

# Display final table using tabulate
print("Premier League Table (season 2024/2025):")
print(tabulate(points_df, headers="keys", tablefmt="fancy_grid"))
print(f"\nResults updated: {latest_game_date.strftime('%Y-%m-%d')}")
