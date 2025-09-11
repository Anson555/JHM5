import random

def helper_turn_teams_into_competitors(teams: list[tuple[str, list[tuple[str, bool]]]]) -> list[tuple[str, str, bool]]:
    # competitors: list[tuple[competitor_name, school_name, competitor_is_seed]]
    competitors = []
    for i in range(len(teams)):
        sch = teams[i][0]
        for j in range(len(teams[i][1])):
            student: tuple[str, bool] = teams[i][1][j]
            competitors.append((student[0], sch, student[1]))
    return competitors

    if (user_input == F):
        return Fals
    print("--警告: 輸入不符合T/F")
    raise ValueError("輸入不符合T/F")

def helper_get_seeds_from_competitors(competitors: list[tuple[str, str, bool]]) -> list[tuple[str, str, bool]]:
    seeds = []
    for i in range(len(competitors)):
        if competitors[i][2] == True:
            seeds.append(competitors[i])
    return seeds

def helper_remove_seeds_from_competitors(competitors: list[tuple[str, str, bool]]) -> list[tuple[str, str, bool]]:
    not_seed = []
    for i in range(len(competitors)):
        if competitors[i][2] == False:
            not_seed.append(competitors[i])
    return not_seed

def helper_remove_same_sch_from_competitors(competitors: list[tuple[str, str, bool]], sch_name: str) -> list[tuple[str, str, bool]]:
    not_same_sch = []
    for i in range(len(competitors)):
        if competitors[i][1] != sch_name:
            not_same_sch.append(competitors[i])
    return not_same_sch

def helper_competitor_to_string(competitor: tuple[str, str, bool]) -> str:
    if competitor == None:
        return "None"
    
    if competitor[2] == True:
        seed_str = "是"
    else:
        seed_str = "否"
    return f"名:{competitor[0]} 學校:{competitor[1]}, 種子:{seed_str}"

def helper_matches_to_string(matches: list[tuple[tuple[str, str, bool], tuple[str, str, bool]]]) -> str:
    s = ""
    if (matches[0][1] == None): # 輪空
        s = s + f"本輪比賽共有{len(matches)*2-1}名參賽者\n"
        s = s + (
            "  本輪有人輪空\n" + ' ' * 4 + "輪空者:\n" 
            + ' ' * 4 + helper_competitor_to_string(matches[0][0]) + "\n"
        )
        j = 1
    else:
        s = s + f"本輪比賽共有{len(matches)*2}名參賽者\n"
        j = 0

    for i in range(j, len(matches)):
        s = s + ("  場次" + str(i+1-j) + ":\n"
            "    / 選手A " + helper_competitor_to_string(matches[i][0]) + " \n"
            "    \\ 選手B " + helper_competitor_to_string(matches[i][1]) + " \n"
        )
    return s

def helper_competitors_to_string(competitors: list[tuple[str, str, bool]]):
    s = ""
    for i in range(len(competitors)):
        s = s + "  " + helper_competitor_to_string(competitors[i]) + "\n"
    return s

def create_teams() -> list[tuple[str, list[tuple[str, bool]]]]:
    # 創建參賽學校和學生列表
    teams = []
    num = int(input("輸入參賽學校數量："))  # 輸入學校數量

    for i in range(num):
        print(f"\n輸入第{i+1}所學校的資料:\n")
        school_name = input(f"  輸入第{i+1}所學校的名稱：")  # 輸入學校名稱
        students = []
        s = ""
        
        print ("每所學校輸入兩名學生的名字")
        for j in range(2):
            student_name = input(' ' * 4 + f"輸入這所學校的 第{j+1}名參賽者 的姓名：")
            s = input(' ' * 4 + f"這名參賽者是否種子選手? (y:是/n:否): ")
            student_is_seed = helper_turn_input_into_bool(s, "y", "n", False)
            students.append((student_name, student_is_seed))
        
        teams.append((school_name, students))  # 存儲學校名稱和學生名單
    return teams # list[tuple[sch_name, list[tuple[student_name, student_is_seed]]]]


def generate_matches(
        competitors: list[tuple[str, str, bool]]
    ) -> list[tuple[tuple[str, str, bool], tuple[str, str, bool]]]:
    # 生成比賽對陣表
    matches = []
    # 處理選手人數為單數的情況
    if (len(competitors) % 2) == 1: 
        seeds = helper_get_seeds_from_competitors(competitors)
        if (len(seeds) == 0): # 沒有種子
            # random選一個輪空
            first = random.choice(competitors)
        else:
            # 種子優先輪空
            first = random.choice(seeds)
        matches.append((first, None))
        competitors.remove(first)

    # 2個參賽者組為對手
    while len(competitors) > 0:
        # 拿出一組中的第一個人 並將他從候選中删掉
        first = competitors[0]
        competitors.pop(0)
        can_be_opponent = competitors.copy()

        if first[2] == True: # 是種子
            # 盡量避免種子與種子交手
            no_seeds = helper_remove_seeds_from_competitors(competitors)
            if len(no_seeds) == 0:
                # 全世界都是種子
                # 唔使改can_be_opponent
                pass
            else:
                can_be_opponent = no_seeds

        # 無論種子或非種子
        # 盡量避免同一學校學生交手
        no_same_sch = helper_remove_same_sch_from_competitors(can_be_opponent, first[1])
        if len(no_same_sch) == 0:
            # 全世界都同學校
            # 唔使改can_be_opponent
            pass
        else:
            can_be_opponent = no_same_sch
        
        # 從can_be_opponent中random選擇一位對手
        second = random.choice(can_be_opponent)
        competitors.remove(second) # 從未選者删除

        matches.append((first, second))

    return matches

def fight(matches: list[tuple[tuple[str, str, bool], tuple[str, str, bool]]]) -> list[tuple[str, str, bool]]:
    winners_of_this_round = []
    for i in range(len(matches)):
        if matches[i][1] == None: # 輪空
            winners_of_this_round.append(matches[i][0])
        else:
            winners_of_this_round.append(random.choice(matches[i]))
    return winners_of_this_round

def print_this_round_result(competitors):
    print(f"本輪幸存{len(competitors)}個選手:\n" + helper_competitors_to_string(competitors))

# main

print("單循環賽制管理系統\n")
teams = create_teams()  # 創建學校和學生
competitors = helper_turn_teams_into_competitors(teams)
i = 0
print("\n")
while True:
    matches = generate_matches(competitors)  # 生成對陣
    print(f"第 {i+1} 輪比賽共有 {len(matches)} 場:")
    print(helper_matches_to_string(matches) + "\n")
    input("按enter 顯示本輪結果...\n")
    competitors = fight(matches)
    print_this_round_result(competitors)
    if (len(competitors) == 1):
        break
    input("按enter 開始下一輪...\n")
    i = i + 1
print("winner is " + helper_competitor_to_string(competitors[0]) + "\n")
