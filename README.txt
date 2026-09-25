BILLBOARD YEAR-END LYRICS SCRAPER + ANALYSIS
Notebook: hot100-scraper.ipynb
==========================================================

WHAT THIS NOTEBOOK DOES
-----------------------
1. Scrapes the Billboard Year-End Hot 100 songs list for a year you type in.
2. Looks up each song on Genius and saves all the lyrics to <year>.lyrics.txt
3. Cleans the lyrics files (removes [Chorus]/(ad-lib) tags, blank lines)
   and builds a word cloud for each year.
4. Runs VADER sentiment analysis on each year and plots a stacked bar chart
   of positive / neutral / negative lines.


WHAT YOU NEED
-------------
- Python 3.9+ (the notebook was written on Python 3.11)
- Jupyter Notebook or JupyterLab
- An internet connection
- A free Genius API token (see Step 3)


STEP 1 - INSTALL PYTHON PACKAGES
--------------------------------
Open a terminal:
  Windows: search "Command Prompt" or "PowerShell" in the Start menu
           (Anaconda Prompt also works if you use Anaconda)
  Mac:     open Terminal (search it with Spotlight, Cmd+Space)

Then run:

    pip install jupyter pandas requests beautifulsoup4 lyricsgenius matplotlib nltk wordcloud

On Mac, if that gives a "command not found: pip" error, use pip3 instead:

    pip3 install jupyter pandas requests beautifulsoup4 lyricsgenius matplotlib nltk wordcloud

Optional but recommended: install into a virtual environment so these
packages don't clash with anything else on your machine.

    python -m venv venv

Activate it:
  Windows (Command Prompt):   venv\Scripts\activate
  Windows (PowerShell):       venv\Scripts\Activate.ps1
  Mac / Linux:                source venv/bin/activate

Your terminal prompt will show (venv) at the start of the line once it's
active. Run the pip install command again after activating.

If you'd rather install from inside Jupyter, run this in a cell instead:

    %pip install pandas requests beautifulsoup4 lyricsgenius matplotlib nltk wordcloud

Then restart the kernel (Kernel > Restart).


STEP 2 - PUT THE NOTEBOOK IN A WORKING FOLDER
---------------------------------------------
Make a folder (e.g. billboard_project) and put the .ipynb file in it.
All output files (lyrics .txt files and word cloud .png files) are saved in
the same folder the notebook runs from, and later cells read them from there
— don't move the notebook mid-run.

Launch Jupyter from that folder:

  Windows:
    cd path\to\billboard_project
    jupyter notebook

  Mac:
    cd path/to/billboard_project
    jupyter notebook

(Slash direction differs: backslash \ on Windows, forward slash / on Mac.)

This opens a browser tab with the Jupyter file browser. Click
hot100-scraper.ipynb to open it.


STEP 3 - GET A GENIUS API TOKEN
-------------------------------
1. Go to https://genius.com/api-clients and sign in / create an account.
2. Click "New API Client". Any app name and website URL is fine.
3. Copy the "Client Access Token".
4. In the FIRST code cell, set:

       GENIUS_API_TOKEN = "your-token-here"

   SECURITY NOTE: Treat this token like a password. Don't share the notebook
   with a live token pasted in it (GitHub, email, class submission, etc.).

   Safer option — keep it out of the notebook entirely:

       import os
       GENIUS_API_TOKEN = os.environ["GENIUS_API_TOKEN"]

   and set the environment variable before launching Jupyter:

     Windows (Command Prompt):
       setx GENIUS_API_TOKEN "your-token-here"
       (close and reopen the terminal for it to take effect)

     Windows (PowerShell):
       [System.Environment]::SetEnvironmentVariable("GENIUS_API_TOKEN","your-token-here","User")
       (close and reopen PowerShell)

     Mac / Linux:
       export GENIUS_API_TOKEN="your-token-here"
       (lasts only for that terminal session — add it to ~/.zshrc or
       ~/.bash_profile to persist across restarts)


STEP 4 - RUN THE CELLS IN ORDER
-------------------------------
CELL 1 (scraper + lyrics downloader)
  - Run it. When prompted "Enter the year:", type a year such as 2025 and
    press Enter.
  - It prints the Billboard URL, the song list, then searches Genius for each
    song. There's a 2-second pause between songs, so a full 100-song list
    takes roughly 5-10 minutes. Let it finish.
  - Output: <year>.lyrics.txt   (example: 2025.lyrics.txt)
  - REPEAT this cell once per year you want to analyze. The analysis cells
    below are set to 2023, 2024, and 2025, so run Cell 1 three times
    (entering 2023, then 2024, then 2025) to have all three files. Each
    rerun overwrites that year's file.

CELL 2 (manual lookup for mismatched titles)
  - Use this when Genius styles a song's title or artist differently than
    Billboard does, so Cell 1 can't find it automatically.
  - Enter the title and artist the way Genius styles them:
       song = genius.search_song([song], [artist])
  - Only works after Cell 1 has been run, since it reuses the `genius`
    object created there.

CELL 3 (clean lyrics + word clouds)
  - Downloads NLTK's stopword list automatically on the first run.
  - Reads the 2023/2024/2025 .lyrics.txt files, cleans them, and writes
    <year>.lyrics.cleaned.txt for each.
  - Shows a word cloud for each year.
  - To change the years, edit this line:   years = ["2023", "2024", "2025"]

CELL 4 (sentiment analysis)
  - Downloads NLTK's VADER lexicon automatically on the first run.
  - Reads the .lyrics.cleaned.txt files from Cell 3 (so Cell 3 must run
    first) and prints the % positive / neutral / negative lines per year.
  - Draws a stacked bar chart comparing years.
  - Has its own copy of the years list:   years = ["2023", "2024", "2025"]
    — keep it matching Cell 3.

CELL 5
  - Empty. Nothing to run.


EXPECTED OUTPUT FILES
---------------------
  2023.lyrics.txt, 2024.lyrics.txt, 2025.lyrics.txt          (raw lyrics)
  2023.lyrics.cleaned.txt, 2024..., 2025...                  (cleaned lyrics)
  <year>_wordcloud_lyrics.png                                (see note below)


KNOWN QUIRKS / TROUBLESHOOTING
------------------------------
"FileNotFoundError: 2023.lyrics.txt"
  You haven't run Cell 1 for that year yet, or Jupyter is running from a
  different folder. Run Cell 1 for each year, and check where the notebook
  is running with:  import os; print(os.getcwd())

"ModuleNotFoundError: No module named ..."
  A package is missing. Re-run the pip install command from Step 1, and make
  sure Jupyter is using the same Python environment you installed into
  (restart the kernel after installing).

"command not found: pip" (Mac)
  Use pip3 and python3 instead of pip and python.

Scraper returns 0 songs, or the song/artist counts don't match
  Billboard may have changed its page layout, or it's blocking requests.
  The cell prints a WARNING when counts differ. The selectors it depends on
  are the "c-title", "c-label", and "o-chart-results-list__item" classes.

Genius can't find a song, or finds the wrong version
  Use Cell 2 to search manually with the title/artist exactly as Genius
  lists them.

"Error fetching lyrics ... 401" or "403"
  Your Genius token is missing or wrong. Re-check Step 3.

"Lyrics not found for ..."
  Normal for a handful of songs (Genius matched the wrong page, or no
  lyrics container was found). Those songs are skipped.

Cell 1 asks for the year again after "Restart & Run All"
  Expected — it uses input(). Just type the year each time.

Word cloud PNG files come out blank
  In Cell 3, plt.savefig(...) is called AFTER plt.show(), which saves an
  empty figure. Move the savefig line so it runs BEFORE plt.show() to fix.

Long run time
  The 2-second time.sleep(2) between songs avoids Genius rate limits.
  Don't remove it.


QUICK START (TL;DR)
-------------------
1. pip install jupyter pandas requests beautifulsoup4 lyricsgenius matplotlib nltk wordcloud
   (Mac: use pip3 if pip alone doesn't work)
2. Put your Genius token in the first cell (or an environment variable).
3. jupyter notebook  ->  open hot100-scraper.ipynb
4. Run Cell 1 three times (enter 2023, 2024, 2025).
5. Use Cell 2 for any songs Genius couldn't match automatically.
6. Run Cell 3, then Cell 4.