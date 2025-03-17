# Zambia Premier League Title Winners Visualization Project

This project visualizes the dynamic history of the Zambia Premier League by animating the cumulative title wins for each team over the years. The animated bar chart race showcases the changing landscape of Zambian football dominance.

## Project Goal

This project aimed to create an animated bar chart race visualizing the Zambia Premier League title wins over the years. The chart would show the cumulative number of titles each team has won, with the bars representing the teams and their lengths corresponding to the title count. The animation would progress year by year, showing how the rankings and title counts change over time. The final output would be a GIF that I could embed in my portfolio.

## Initial Setup and Data

I started by manually gathering the data, which is publicly available on Wikipedia: [https://en.wikipedia.org/wiki/Zambia_Super_League](https://en.wikipedia.org/wiki/Zambia_Super_League).

The data initially consisted of the year and the winner of the Zambia Premier League for each year. I cleaned and organized this data in a Google Sheet here: [https://docs.google.com/spreadsheets/d/1A02C4joz0Xx6UMahAHPXz2jBgNYF8orp2RcxhfiCggs/edit?gid=2147110241#gid=2147110241](https://docs.google.com/spreadsheets/d/1A02C4joz0Xx6UMahAHPXz2jBgNYF8orp2RcxhfiCggs/edit?gid=2147110241#gid=2147110241)

I then loaded this data into a Pandas DataFrame in my Google Colab notebook.

## First Attempt (Basic Animation)

My initial attempt (below) focused on creating a basic animation. I used matplotlib to create the bar charts for each year and imageio to combine these charts into a GIF. However, the animation was too fast, and all teams appeared on the chart from the beginning, even before they had won any titles.

```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import imageio
import os

# Your data
data = {'Year': list(range(1962, 2025)),
        'Winners': ['Roan United (Luanshya)', 'Mufulira Wanderers (Mufulira)', 'City of Lusaka (Lusaka)',
                    'Mufulira Wanderers (Mufulira)', 'Mufulira Wanderers (Mufulira)', 'Mufulira Wanderers (Mufulira)',
                    'Kabwe Warriors (Kabwe)', 'Mufulira Wanderers (Mufulira)', 'Kabwe Warriors (Kabwe)', 'Kabwe Warriors (Kabwe)',
                    'Kabwe Warriors (Kabwe)', 'Zambia Army (Lusaka)', 'Zambia Army (Lusaka)', 'Green Buffaloes (Lusaka)',
                    'Mufulira Wanderers (Mufulira)', 'Green Buffaloes (Lusaka)', 'Mufulira Wanderers (Mufulira)',
                    'Green Buffaloes (Lusaka)', 'Nchanga Rangers (Chingola)', 'Green Buffaloes (Lusaka)', 'Nkana (Kitwe)',
                    'Nkana (Kitwe)', 'Power Dynamos (Kitwe)', 'Nkana (Kitwe)', 'Nkana (Kitwe)', 'Kabwe Warriors (Kabwe)',
                    'Nkana (Kitwe)', 'Nkana (Kitwe)', 'Nkana (Kitwe)', 'Power Dynamos (Kitwe)', 'Nkana (Kitwe)', 'Nkana (Kitwe)',
                    'Power Dynamos (Kitwe)', 'Mufulira Wanderers (Mufulira)', 'Mufulira Wanderers (Mufulira)',
                    'Power Dynamos (Kitwe)', 'Nchanga Rangers (Chingola)', 'Nkana (Kitwe)', 'Power Dynamos (Kitwe)',
                    'Nkana (Kitwe)', 'Zanaco', 'Zanaco', 'Red Arrows (Lusaka)', 'Zanaco', 'Zanaco', 'ZESCO United (Ndola)',
                    'ZESCO United (Ndola)', 'Zanaco', 'ZESCO United (Ndola)', 'Power Dynamos (Kitwe)', 'Zanaco', 'Nkana (Kitwe)',
                    'ZESCO United (Ndola)', 'ZESCO United (Ndola)', 'Zanaco', 'ZESCO United (Ndola)', 'ZESCO United (Ndola)',
                    'ZESCO United (Ndola)', 'Nkana (Kitwe)', 'ZESCO United (Ndola)', 'Red Arrows (Lusaka)', 'Power Dynamos (Kitwe)',
                    'Red Arrows (Lusaka)']}

df = pd.DataFrame(data)

# Calculate title counts
title_counts = df.groupby('Winners').size().reset_index(name='Count')

# Store cumulative counts for each team
cumulative_counts = {winner: 0 for winner in title_counts['Winners']}

# Prepare data for animation
animated_data = []
for year, winner in zip(df['Year'], df['Winners']):
    cumulative_counts[winner] += 1
    temp_df = title_counts.copy()
    for w in temp_df['Winners']:
        temp_df.loc[temp_df['Winners'] == w, 'Count'] = cumulative_counts[w]
    temp_df['Year'] = year
    animated_data.append(temp_df)

animated_df = pd.concat(animated_data)

# Function to create bar chart
def create_bar_chart(df, year):
    df_year = df[df['Year'] == year].sort_values('Count', ascending=False)

    fig, ax = plt.subplots(figsize=(10, 6))
    ax.clear()

    bars = ax.barh(df_year['Winners'], df_year['Count'], color='skyblue')

    ax.set_title(f"Zambia League Titles Over the Years ({year})", fontsize=14)
    ax.set_xlabel("Title Count", fontsize=12)
    ax.invert_yaxis()
    ax.xaxis.set_major_formatter(ticker.StrMethodFormatter('{x:,.0f}'))
    ax.grid(axis='x', linestyle='--', alpha=0.7)

    for bar in bars:
        ax.text(bar.get_width() + 0.2, bar.get_y() + bar.get_height()/2,
                f'{bar.get_width():.0f}', va='center', ha='left', fontsize=9)

    return fig

# Generate frames
years = animated_df['Year'].unique()
filenames = []

for year in years:
    fig = create_bar_chart(animated_df, year)
    filename = f'frame_{year}.png'
    fig.savefig(filename)
    filenames.append(filename)
    plt.close(fig)

# **Set frame duration and duplicate each frame 20 times to slow down the transition**
gif_duration_per_frame = 3  # Frame duration (seconds)
duplicate_per_frame = 20    # Repeat each frame 20 times

# Create GIF
gif_path = 'zambia_league_titles_with_duplicates.gif'
with imageio.get_writer(gif_path, mode='I', duration=gif_duration_per_frame) as writer:
    for filename in filenames:
        image = imageio.v2.imread(filename)
        for _ in range(duplicate_per_frame):  # Duplicate each frame 20 times
            writer.append_data(image)

# Clean up image files
for filename in set(filenames):
    os.remove(filename)

# Display the GIF (if running in Jupyter Notebook)
try:
    from IPython.display import display, Image
    display(Image(filename=gif_path))
except ImportError:
    print("GIF created successfully! Open 'zambia_league_titles_with_duplicates.gif' to view.")



