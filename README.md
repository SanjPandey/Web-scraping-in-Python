import requests
from bs4 import BeautifulSoup
import pandas as pd

# Cricbuzz scorecard URL
url = "https://www.cricbuzz.com/live-cricket-scorecard/118541/uae-vs-ban-1st-t20i-bangladesh-tour-of-uae-2025"
headers = {"User-Agent": "Mozilla/5.0"}
response = requests.get(url, headers=headers)
soup = BeautifulSoup(response.content, "html.parser")

# Get all innings blocks
innings_blocks = soup.find_all("div", class_="cb-col cb-col-100 cb-ltst-wgt-hdr")

for i, block in enumerate(innings_blocks):
    title_tag = block.find("span")
    if not title_tag:
        continue  # Skip if no innings title found

    innings_name = title_tag.get_text(strip=True)
    print(f"\n {innings_name}")

    rows = block.find_all("div", class_="cb-col cb-col-100 cb-scrd-itms")

    # --- Batting Records ---
    batting_data = []
    for row in rows:
        cols = row.find_all("div", recursive=False)
        cols_text = [c.get_text(strip=True) for c in cols if c.text.strip()]
        if len(cols_text) == 7 and cols_text[0].lower() not in ["extras", "total"]:
            batting_data.append(cols_text)

    if batting_data:
        df_bat = pd.DataFrame(batting_data, columns=["Batter", "Dismissal", "R", "B", "4s", "6s", "SR"])
        file_name_bat = f"{innings_name.replace(' ', '_')}_batting.csv"
        df_bat.to_csv(file_name_bat, index=False)
        print(f" Batting data saved: {file_name_bat}")
        print(df_bat)
    else:
        print(" No batting data found.")

    # --- Bowling Records (usually in next block) ---
    if i + 1 < len(innings_blocks):
        next_block = innings_blocks[i + 1]
        bowl_rows = next_block.find_all("div", class_="cb-col cb-col-100 cb-scrd-itms")

        bowling_data = []
        for row in bowl_rows:
            cols = row.find_all("div", recursive=False)
            cols_text = [c.get_text(strip=True) for c in cols if c.text.strip()]
            if len(cols_text) == 8:
                bowling_data.append(cols_text)

        if bowling_data:
            df_bowl = pd.DataFrame(bowling_data, columns=["Bowler", "O", "M", "R", "W", "NB", "WD", "ECO"])
            file_name_bowl = f"{innings_name.replace(' ', '_')}_bowling.csv"
            df_bowl.to_csv(file_name_bowl, index=False)
            print(f" Bowling data saved: {file_name_bowl}")
            print(df_bowl)
        else:
            print(" No bowling data found.")


OutPut:

 Bangladesh Innings
 Saved: Bangladesh_Innings.csv
                Batter                           Dismissal    R   B 4s 6s  \
0         Tanzid Hasan     c Rahul Chopra b Matiullah Khan   10   9  1  1   
1  Parvez Hossain Emon                        b Jawadullah  100  54  5  9   
2       Litton Das (c)                    lbw b Jawadullah   11   8  0  1   
3        Towhid Hridoy  c Muhammad Zohaib b Dhruv Parashar   20  15  1  1   
4         Mahedi Hasan         c Rahul Chopra b Jawadullah    2   5  0  0   
5       Jaker Ali (wk)  c Sanchit Sharma b Muhammad Zuhaib   13  14  0  1   
6       Shamim Hossain                    lbw b Jawadullah    6   5  1  0   
7   Tanzim Hasan Sakib                             not out    7   9  1  0   
8         Tanvir Islam                             not out    1   4  0  0   

       SR  
0  111.11  
1  185.19  
2  137.50  
3  133.33  
4   40.00  
5   92.86  
6  120.00  
7   77.78  
8   25.00  

 United Arab Emirates Innings
 Saved: United_Arab_Emirates_Innings.csv
                 Batter                            Dismissal   R   B 4s 6s  \
0       Muhammad Zohaib         c (sub)Shanto b Hasan Mahmud   9   9  2  0   
1   Muhammad Waseem (c)    c Mustafizur b Tanzim Hasan Sakib  54  39  7  2   
2       Alishan Sharafu             c Jaker Ali b Mustafizur   1   3  0  0   
3     Rahul Chopra (wk)  c Mahedi Hasan b Tanzim Hasan Sakib  35  22  5  1   
4             Asif Khan       c Towhid Hridoy b Hasan Mahmud  42  21  3  4   
5        Dhruv Parashar         c (sub)Shanto b Tanvir Islam   3   7  0  0   
6        Sanchit Sharma          c Litton Das b Hasan Mahmud   4   5  0  0   
7       Muhammad Zuhaib          c Tanzid Hasan b Mustafizur   0   2  0  0   
8            Haider Ali         c (sub)Shanto b Mahedi Hasan   0   3  0  0   
9        Matiullah Khan                              not out   0   4  0  0   
10  Muhammad Jawadullah          st Jaker Ali b Mahedi Hasan   6   5  1  0   

        SR  
0   100.00  
1   138.46  
2    33.33  
3   159.09  
4   200.00  
5    42.86  
6    80.00  

output
Batting Scorecard							
Batter	Dismissal	R	B	4s	6s	SR	
Tanzid Hasan	c Rahul Chopra b Matiullah Khan	10	9	1	1	111.11	
Parvez Hossain Emon	b Jawadullah	100	54	5	9	185.19	
Litton Das (c)	lbw b Jawadullah	11	8	0	1	137.50	
Towhid Hridoy	c Muhammad Zohaib b Dhruv Parashar	20	15	1	1	133.33	
Mahedi Hasan	c Rahul Chopra b Jawadullah	2	5	0	0	40.00	
Jaker Ali (wk)	c Sanchit Sharma b Muhammad Zuhaib	13	14	0	1	92.86	
Shamim Hossain	lbw b Jawadullah	6	5	1	0	120.00	
Tanzim Hasan Sakib	not out	7	9	1	0	77.78	
Tanvir Islam	not out	1	4	0	0	25.00	
							
Extras 21 (b 4, lb 4, w 10, nb 3, p 0)							
Total 191 (7 wkts, 20 Ov)							
							
Bowling Scorecard							
Bowler	O	M	R	W	NB	WD	ECO
Dhruv Parashar	4	0	32	1	0	2	8.00
Matiullah Khan	4	0	46	1	3	2	11.50
Haider Ali	4	0	30	0	0	0	7.50
Jawadullah	4	0	21	4	0	5	5.20
Sanchit Sharma	1	0	18	0	0	0	18.00
Muhammad Zuhaib	3	0	36	1	0	1	12.00


7     0.00  
8     0.00  
9     0.00  
10  120.00  

            
