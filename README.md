using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Threading;

internal class Program
{
    private static Character player;

    static bool isArmorEquipped = true; // 무쇠갑옷 착용한 상태로 시작

    
    static void Main(string[] args)
    {
       
        GameDataSetting();
        DisplayGameIntro();

    }
    

    static void DisplayGameIntro()
    {
        Console.Clear();

        Console.WriteLine("스파르타 마을에 오신 여러분 환영합니다.");
        Console.WriteLine("이곳에서 던전으로 들어가기 전 활동을 할 수 있습니다.\n");
        Console.WriteLine("1. 상태 보기\n");
        Console.WriteLine("2. 인벤토리\n");
        Console.WriteLine("3. 상점 가기\n");
        Console.WriteLine("4. 던전으로 들어가기\n");
        Console.Write("원하시는 행동을 입력해주세요. >> ");

        string action = Console.ReadLine();

        switch (action)
        {
            case "0":
               
            case "1":
                DisplayMyinfo();
                break;
            case "2":
                DisplayInventory();
                break;
            case "3":
                DisplayShop();
                break;
            case "4":
                평온한평야();
                break;

            default:
                Console.WriteLine("잘못된 입력입니다. 다시 시도해주세요.");
                DisplayGameIntro();
                break;
        }
    }
    static int CheckValidInput(int min, int max)
    {
        while (true)
        {
            string input = Console.ReadLine();

            bool parseSuccess = int.TryParse(input, out var ret);
            if (parseSuccess)
            {
                if (ret >= min && ret <= max)
                    return ret;
            }

            Console.WriteLine("잘못된 입력입니다.");
        }
    }
    static void GameDataSetting()
    {
       
        //캐릭터 정보
        player = new Character("이방인", "전사", 1, 10, 5, 100, 1500);
    }

    static void DisplayMyinfo()
    {
        Console.Clear();

        


        Console.WriteLine("상태보기");
        Console.WriteLine("캐릭터의 정보를 표시합니다.");
        Console.WriteLine();
        Console.WriteLine($"Lv.{player.Level}");
        Console.WriteLine($"{player.Name}({player.Job})");
        Console.WriteLine($"공격력: {player.Atk} + {player.EquippedWeapon.AttackBonus + player.EquippedArmor.AttackBonus}");
        Console.WriteLine($"방어력 : {player.Def} + {player.EquippedArmor.DefenseBonus + player.EquippedWeapon.DefenseBonus} ");
        Console.WriteLine($"체력 : {player.Hp}");
        Console.WriteLine($"Gold : {player.Gold} G");
        Console.WriteLine();
        Console.WriteLine("0. 나가기");

        int input = CheckValidInput(0, 0);
        switch (input)
        {
            case 0:
                DisplayGameIntro();
                break;
        }
    }

    static void DisplayInventory()
    {
        Console.Clear();

        Console.WriteLine("인벤토리");
        Console.WriteLine("보유 중인 아이템을 관리할 수 있습니다.\n");
        Console.WriteLine("[아이템 목록]");
        foreach (var item in Inventory.Items)
        {
            Console.WriteLine($"- {(item == Inventory.EquippedArmor || item == Inventory.EquippedWeapon ? "[E]" : "")}{item.Name} | 공격력 +{item.AttackBonus} | 방어력 +{item.DefenseBonus} | {item.Description}");
        }
        Console.WriteLine("1. 장착 관리");
        Console.WriteLine("0. 나가기");

        Console.Write("원하시는 행동을 입력해주세요. >> ");

        int input = CheckValidInput(0, 1);
        switch (input)
        {
            case 0:
                DisplayGameIntro();
                break;
            case 1:
                ManageEquip();
                break;
            default:
                Console.WriteLine("잘못된 입력입니다. 다시 시도해주세요.");
                break;

        }
    }

    /// <summary>
    /// //////////캐릭터 구간
    /// </summary>
    public class Character
    {
        public string Name { get; }
        public string Job { get; }
        public int Level { get; }
        public int Atk { get; }
        public int Def { get; }
        public int Hp { get; private set; }
        public void DecreaseHp(int amount)
        {
            if (amount < 0)
                throw new ArgumentException("감소시킬 체력은 양수여야 합니다.");

            Hp -= amount;
            if (Hp < 0)
                Hp = 0;  // 체력이 0보다 작아지지 않도록
        }

        // 체력을 증가시키는 메서드 (예: 회복 아이템 사용 시)
        public void IncreaseHp(int amount)
        {
            if (amount < 0)
                throw new ArgumentException("증가시킬 체력은 양수여야 합니다.");

            Hp += amount;
            // 만약 체력의 최대치를 정하려면, 체력이 그 값 이상으로 증가하지 않도록 제한할 수 있습니다.
        }
        private int _gold;
        public int Gold
        {
            get { return _gold; }
            private set { _gold = value; }
        }
        public void DeductGold(int amount)
        {
            if (amount < 0) // 골드를 추가할 때
            {
                _gold -= amount; // amount는 음수이므로, _gold에서 뺀다면 _gold가 증가합니다.
                return;
            }

            if (_gold >= amount) // 직접 필드에 접근하여 값을 변경
            {
                _gold -= amount;
            }
            else
            {
                throw new InvalidOperationException("골드가 충분하지 않습니다!");
            }
        }


        public Character(string name, string job, int level, int atk, int def, int hp, int gold)
        {
            Name = name;
            Job = job;
            Level = level;
            Atk = atk;
            Def = def;
            Hp = hp;
            _gold = gold;
        }
        public Item EquippedArmor => Inventory.EquippedArmor;
        public Item EquippedWeapon => Inventory.EquippedWeapon;
       
    }

    public class Item
    {
        public string Name { get; }
        public int AttackBonus { get; }

        public int DefenseBonus { get; }

        public string Description { get; }
        public int Price { get; set; }

        public Item(string name, int attackBonus, int defenseBonus, string description, int price)
        {
            Name = name;
            AttackBonus = attackBonus;
            DefenseBonus = defenseBonus;
            Description = description;
            Price = price;
        }
    }

    /// <summary>
    /// ////////// 인벤토리 구간
    /// </summary>
    public static class Inventory
    {
        public static List<Item> Items { get; private set; } = new List<Item>
        { //아이템 만들기
            new Item("무쇠갑옷", 0, 5, "무쇠로 만들어져 튼튼한 갑옷입니다.", 300),
            new Item("낡은 검", 2, 0, "쉽게 볼 수 있는 낡은 검입니다.", 300)
        };
        public static Item EquippedArmor { get; private set; } = Items[0];
        public static Item EquippedWeapon { get; private set; } = Items[1];
        public static Item None = new Item("None", 0, 0, "아무 것도 장착하지 않았습니다.", 0);

        public static void EquipArmor(Item armor)
        {
            EquippedArmor = armor;
        }

        public static void EquipWeapon(Item weapon)
        {
            EquippedWeapon = weapon;
        }

        public static void UnEquipArmor()
        {
            EquippedArmor = None;
        }
        public static void UnEquipWeapon()
        {
            EquippedWeapon = None;
        }

    }

    /// <summary>
    /// ///////////장착 구간
    /// </summary>

    static void ManageEquip()
    {
        Console.WriteLine("장착/해제하실 아이템을 선택해주세요.");

        for (int i = 0; i < Inventory.Items.Count; i++)
        {
            Console.WriteLine($"{i + 1}.{Inventory.Items[i].Name}");
        }
        Console.WriteLine($"{Inventory.Items.Count + 1}. 갑옷 해제");
        Console.WriteLine($"{Inventory.Items.Count + 2}. 무기 해제");
        Console.WriteLine("0.나가기");

        int equipAction = CheckValidInput(0, Inventory.Items.Count + 2);

        if (equipAction == 0)
        {
            DisplayInventory();
            return;
        }

        if (equipAction == Inventory.Items.Count + 1)
        {
            Inventory.UnEquipArmor();
            Console.WriteLine("갑옷을 해제하였습니다.");
            DisplayInventory();
            return;
        }

        else if (equipAction == Inventory.Items.Count + 2)
        {
            Inventory.UnEquipWeapon();
            Console.WriteLine("무기를 해제하였습니다.");
            DisplayInventory();
            return;
        }

        else
        {

            var selectedItem = Inventory.Items[equipAction - 1];

            if (selectedItem.AttackBonus > 0)
            {
                Inventory.EquipWeapon(selectedItem);
                Console.WriteLine($"{selectedItem.Name}을(를) 장착하였습니다.");
                DisplayInventory();
            }
            else if (selectedItem.DefenseBonus > 0)
            {
                Inventory.EquipArmor(selectedItem);
                Console.WriteLine($"{selectedItem.Name}을(를) 장착하였습니다.");
                DisplayInventory();
            }
        }
        

    }

    /// <summary>
    /// /////////////// 상점구간
    /// </summary>
    static void DisplayShop()
    {
        Console.Clear();
        Console.WriteLine("상점에 오신 것을 환영합니다!");
        Console.WriteLine("[상품 목록]");

        for (int i = 0; i < Shop.ItemsForSale.Count; i++)
        {
            var item = Shop.ItemsForSale[i];
            Console.WriteLine($"{i + 1}. {item.Name} | 가격: {item.Price}G | 공격력 +{item.AttackBonus} | 방어력 +{item.DefenseBonus}");
        }
        Console.WriteLine($"{Shop.ItemsForSale.Count + 1}. 아이템 판매");
        Console.WriteLine("0. 나가기");
        Console.Write("구매하실 아이템 번호를 선택해주세요. >> ");

        int input = CheckValidInput(0, Shop.ItemsForSale.Count + 2);

        if (input == 0)
        {
            DisplayGameIntro();
            return;
        }
        if (input == Shop.ItemsForSale.Count + 1)
        {
            SellItem();
            return;
        }

        BuyItem(Shop.ItemsForSale[input - 1]);
    }
    static void BuyItem(Item item)
    {
        if (player.Gold < item.Price)
        {
            Console.WriteLine("골드가 부족합니다!");
            DisplayShop();
            return;
        }
        
        Inventory.Items.Add(item);
        player.DeductGold(item.Price);
        Console.WriteLine($"{item.Name}을(를) 구매했습니다!");

        DisplayShop();
    }
    static void SellItem()
    {
        Console.Clear();
        Console.WriteLine("판매하실 아이템을 선택해주세요.");

        for (int i = 0; i < Inventory.Items.Count; i++)
        {
            var item = Inventory.Items[i];
            Console.WriteLine($"{i + 1}. {item.Name} | 판매가격: {(int)(item.Price * 0.5)}G | 공격력 +{item.AttackBonus} | 방어력 +{item.DefenseBonus}");
        }

        Console.WriteLine("0. 나가기");
        Console.Write("판매하실 아이템 번호를 선택해주세요. >> ");

        int input = CheckValidInput(0, Inventory.Items.Count + 2);

        switch (input)
        {
            case 0:
                DisplayShop();
                break;

            default:
                var selectedItem = Inventory.Items[input - 1];
                player.DeductGold(-(int)(selectedItem.Price * 0.5));
                Inventory.Items.Remove(selectedItem);
                Console.WriteLine($"{selectedItem.Name}을(를) {(int)(selectedItem.Price * 0.5)}G에 판매했습니다!");
                Console.WriteLine("계속하려면 아무 키나 누르세요.");
                Console.ReadKey();
                DisplayShop();
                break;
        }
    }


    public static class Shop
    {
        public static List<Item> ItemsForSale { get; } = new List<Item>
        {
          new Item("강철 갑옷", 0, 10, "강철로 만들어진 갑옷", 500),
          new Item("빛나는 검", 10, 0, "빛나는 검입니다", 800)
        };
    }
    /// <summary>
    /// //////////////몬스터 구간
    /// </summary>
    public static List<Monster> Monsters = new List<Monster>
    {
         new Monster("얌전한 슬라임", 5, 2, 20),
         new Monster("어린 고블린", 3, 1, 15),
         new Monster("고블린", 10, 5, 50),
         new Monster("와일드 보어", 20, 15, 100),
         new Monster("늑대 무리", 35, 25, 300),
         // 추가로 더 많은 몬스터를 넣을 수 있습니다.
    };
    static Random rng = new Random();

    static Monster GetRandomMonster()
    {
        int randomIndex = rng.Next(Monsters.Count);
        return Monsters[randomIndex];
    }
    static void 평온한평야()
    {
        Console.Clear();
        Console.WriteLine("어두운 던전에 들어왔습니다. 앞에는 몬스터가 보입니다.");

        Monster monster = GetRandomMonster(); // 예시 몬스터


        Console.WriteLine($"{monster.Name}이(가) 공격해 옵니다!");
        Console.WriteLine("1. 공격하기");
        Console.WriteLine("2. 도망가기");

        int choice = CheckValidInput(1, 2);
        if (choice == 1)
        {
            Fight(monster);
        }
        else
        {
            Console.WriteLine("부끄러움을 무릅쓰고 도망갑니다.");
            DisplayGameIntro();
        }
    }
    static void Fight(Monster monster)
    {
        while (player.Hp > 0 && monster.Hp > 0)
        {
            // 예시로 플레이어가 먼저 공격
            int playerDamage = player.Atk + player.EquippedWeapon.AttackBonus - monster.Def;
            monster.Hp -= playerDamage;
            Console.WriteLine($"{player.Name}이(가) {monster.Name}에게 {playerDamage}의 피해를 입혔습니다!");

            if (monster.Hp <= 0) // 몬스터를 쓰러뜨릴 경우
            {
                Console.WriteLine($"{monster.Name}을(를) 쓰러뜨렸습니다!");
                // 보상 주기
                player.DeductGold(-100); // 예: 100골드 획득
                Console.WriteLine("100골드를 획득했습니다!");

                // 플레이어가 아무 키를 누를 때까지 기다림
                Console.WriteLine("계속하려면 아무 키나 누르세요.");
                Console.ReadKey();

                DisplayGameIntro();
                return;
            }

            int monsterDamage = monster.Atk - player.Def;
            player.DecreaseHp(monsterDamage);
            Console.WriteLine($"{monster.Name}이(가) {player.Name}에게 {monsterDamage}의 피해를 입혔습니다!");

            if (player.Hp <= 0) // 플레이어가 쓰러질 경우
            {
                Console.WriteLine("당신은 쓰러졌습니다...");
                // 게임 끝내기 또는 다른 행동
                Environment.Exit(0);
            }
        }
    }
    public class Monster
    {
        public string Name { get; }
        public int Atk { get; }
        public int Def { get; }
        public int Hp { get; set; }

        public Monster(string name, int atk, int def, int hp)
        {
            Name = name;
            Atk = atk;
            Def = def;
            Hp = hp;
        }
    }

}
