팩토리 메서드 패턴, 전략 패턴, 어댑터 패턴을 조합하여 디자인 패턴을 활용한 게임 제작
아래는 외부 라이브러리를 가정한 추가 헤더 파일 Hammer.h

    #pragma once
    #include <iostream>
    using namespace std;
    
    class Hammer {
    public:
        void smash() {
            cout << "Hammer smash!!" << endl;
        }
    };

아래는 디자인 패턴 게임 메인 코드 

    #include <iostream>
    #include <memory>
    #include <random>
    #include <string>
    #include <map>
    #include "Hammer.h"
    
    using namespace std;
    
    int PlayerHP = 1000;
    int PlayerMP = 0;
    int MonsterHP = 1000;
    int mobDamage;
    
    random_device rd;
    mt19937 gen(rd());
    uniform_int_distribution<int> dist(3, 8);
    uniform_int_distribution<int> mob(70, 140);
    uniform_int_distribution<int> crit(0, 99);
    
    class Weapon;
    
    struct AttackResult {
    	int damage = 0;
    	int mpChange = 0;
    	bool critical = false;
    };
    
    // Strategy
    class AttackStrategy {
    public:
    	virtual AttackResult execute(const Weapon& weapon) = 0;
    	virtual const char* getName() = 0;
    	virtual ~AttackStrategy() {}
    };
    
    // Product
    class Weapon {
    	unique_ptr<AttackStrategy> strategy;
    protected:
    	int critChance = 0;
    
    public:
    	void setCritChance(int c) { critChance = c; }
    	int getCritChance() const { return critChance; }
    
    	virtual int getNormalDamage() const = 0;
    	virtual int getChargedDamage() const = 0;
    	virtual int getComboDamage() const = 0;
    	virtual void equip() = 0;
    
    	void SetAttackStrategy(unique_ptr<AttackStrategy> s) {
    		strategy = move(s);
    	}
    
    	void Attack(int& MonsterHP) {
    		if (!strategy) {
    			cout << "아직 공격 전략을 선택하지 못했다..." << endl;
    		}
    
    		AttackResult result = strategy->execute(*this);
    
    		PlayerMP += result.mpChange;
    		if (PlayerMP < 0) PlayerMP = 0;
    
    		int damage = result.damage;
    
    		if (result.critical) {
    			damage = static_cast<int>(damage * 1.2);
    			cout << "크리티컬!! ";
    		}
    
    		MonsterHP -= damage;
    		if (MonsterHP < 0) MonsterHP = 0;
    
    		cout << strategy->getName() << "으로 " << damage << "의 피해를 입혔다!" << endl;
    	}
    	virtual ~Weapon() {}
    };
    
    // ConcreteStrategy
    class NormalAttack : public AttackStrategy {
    public:
    	AttackResult execute(const Weapon& weapon) override {
    		AttackResult res;
    		res.damage = weapon.getNormalDamage();
    		res.mpChange = +1;
    
    		int roll = crit(gen);
    		res.critical = (roll < weapon.getCritChance());
    		return res;
    	}
    
    	const char* getName() override { return "기본공격"; }
    };
    
    
    class ChargedAttack : public AttackStrategy {
    public:
    	AttackResult execute(const Weapon& weapon) override {
    		AttackResult res;
    		res.damage = weapon.getChargedDamage();
    		res.mpChange = -2;
    
    		int roll = crit(gen);
    		res.critical = (roll < weapon.getCritChance());
    		return res;
    	}
    
    	const char* getName() override { return "강공격"; }
    };
    
    
    class ComboAttack : public AttackStrategy {
    public:
    	AttackResult execute(const Weapon& weapon) override {
    		AttackResult res;
    		res.damage = weapon.getComboDamage() * dist(gen);
    		res.mpChange = -3;
    
    		int roll = crit(gen);
    		res.critical = (roll < weapon.getCritChance());
    		return res;
    	}
    
    	const char* getName() override { return "연속공격"; }
    };
    
    
    // ConcreteProduct
    class Sword : public Weapon {
    public:
    	Sword() { setCritChance(3); }
    	int getNormalDamage() const override { return 100; }
    	int getChargedDamage() const override { return 110; }
    	int getComboDamage() const override { return 40; }
    	void equip() override { cout << "당신은 검을 뽑았다!" << endl << endl; }
    };
    
    class Bow : public Weapon {
    public:
    	Bow() { setCritChance(1); }
    	int getNormalDamage() const override { return 25; }
    	int getChargedDamage() const override { return 230; }
    	int getComboDamage() const override { return 100; }
    	void equip() override { cout << "당신은 활 시위를 당겼다!" << endl << endl; }
    };
    
    // Adapter: HammerAdapter
    class HammerAdapter : public Weapon {
    	unique_ptr<Hammer> hammer;
    
    public:
    	HammerAdapter() : hammer(make_unique<Hammer>()) { setCritChance(5); }
    
    	int getNormalDamage() const override {
    		hammer->smash();
    		return 45;
    	}
    	int getChargedDamage() const override {
    		hammer->smash();
    		return 260;
    	}
    	int getComboDamage() const override {
    		hammer->smash();
    		return 55;
    	}
    	void equip() override { cout << "당신은 망치를 들어 올렸다!" << endl << endl; }
    
    	~HammerAdapter() override = default;
    };
    
    // Creator
    class WeaponFactory {
    public:
    	virtual unique_ptr<Weapon> CreateWeapon() = 0;
    	virtual ~WeaponFactory() {}
    };
    
    // ConcreteCreator
    class SwordFactory : public WeaponFactory {
    public:
    	unique_ptr<Weapon> CreateWeapon() override {
    		return make_unique<Sword>();
    	}
    };
    
    class BowFactory : public WeaponFactory {
    public:
    	unique_ptr<Weapon> CreateWeapon() override {
    		return make_unique<Bow>();
    	}
    };
    
    // Adapter 팩토리 (Adapter를 팩토리와 연결)
    class HammerFactory : public WeaponFactory {
    public:
    	unique_ptr<Weapon> CreateWeapon() override {
    		return make_unique<HammerAdapter>();
    	}
    };
    
    int main()
    {
    	int selectWeapon;
    	string atkStrategy;
    
    	SwordFactory SF;
    	BowFactory BF;
    	HammerFactory HF;
    
    	cout << "\\(`し´)/" << endl;
    	cout << "!!!갑자기 몬스터가 나타났다!!!" << endl;
    	cout << "어떤 무기를 사용할까..." << endl;
    	cout << "=========================[ 숫자 입력으로 무기 선택 ]==========================" << endl;
    	cout << "(1) 검  : 모든 공격으로 준수한 데미지를 줄 수 있다. 무난한 선택일 듯 하다" << endl;
    	cout << "(2) 활  : 기본 공격은 매우 약하지만 연속 공격으로 아주 강한 피해를 줄 수 있다." << endl;
    	cout << "(3) 망치: 기본 공격과 강공격을 조합하여 아주 강한 피해를 줄 수 있다." << endl;
    	cout << "==============================================================================" << endl << "> ";
    	cin >> selectWeapon;
    	cout << endl;
    
    	while (cin.fail() || (selectWeapon != 1 && selectWeapon != 2 && selectWeapon != 3)) {
    		cin.clear();
    		cin.ignore(10, '\n');
    		cout << "좋은 선택이 아닌 것 같다. 한번 더 생각해보자..." << endl << "> ";
    		cin >> selectWeapon;
    		cout << endl;
    	}
    
    	unique_ptr<Weapon> weapon;
    
    	map<int, WeaponFactory*> factories = {
    	{1, &SF},
    	{2, &BF},
    	{3, &HF}
    	};
    
    	weapon = factories[selectWeapon]->CreateWeapon();
    
    	weapon->equip();
    
    	while (PlayerHP > 0 && MonsterHP > 0) {
    		cout << "어떻게 공격하는게 좋을까..." << endl;
    		cout << "===========[ Q/W/E 입력으로 공격 전략 선택 ]==========" << endl;
    		cout << "(Q) 기본공격: MP 1 회복 / 비교적 약한 피해를 준다." << endl;
    		cout << "(W) 강공격  : MP 2 소모 / 강한 피해를 줄 수 있다." << endl;
    		cout << "(E) 연속공격: MP 3 소모 / 아주 강한 피해를 줄 수 있다." << endl;
    		cout << "======================================================" << endl << "> ";
    		cin >> atkStrategy;
    		atkStrategy[0] = toupper(atkStrategy[0]);
    		cout << endl;
    
    		while (!(atkStrategy.size() == 1) || (toupper(atkStrategy[0])) != 'Q' && (toupper(atkStrategy[0])) != 'W' && (toupper(atkStrategy[0])) != 'E') {
    			cout << "좋은 선택이 아닌 것 같다. 한번 더 생각해보자..." << endl << "> ";
    			cin >> atkStrategy;
    			atkStrategy[0] = toupper(atkStrategy[0]);
    			cout << endl;
    		}
    
    		switch (atkStrategy[0]) {
    		case 'Q': {
    			weapon->SetAttackStrategy(make_unique<NormalAttack>());
    			weapon->Attack(MonsterHP);
    			if (MonsterHP < 0) MonsterHP = 0;
    			break;
    		}
    		case 'W': {
    			if (PlayerMP >= 2) {
    				weapon->SetAttackStrategy(make_unique<ChargedAttack>());
    				weapon->Attack(MonsterHP);
    				if (MonsterHP < 0) MonsterHP = 0;
    			}
    			else {
    				cout << "MP가 부족하다, 다른 공격 전략을 생각해보자..." << endl << endl;
    				continue;
    			}
    			break;
    		}
    		case 'E': {
    			if (PlayerMP >= 3) {
    				weapon->SetAttackStrategy(make_unique<ComboAttack>());
    				weapon->Attack(MonsterHP);
    				if (MonsterHP < 0) MonsterHP = 0;
    			}
    			else {
    				cout << "MP가 부족하다, 다른 공격 전략을 생각해보자..." << endl << endl;
    				continue;
    			}
    			break;
    		}
    		}
    
    		cout << "몬스터 HP: " << MonsterHP << endl << endl;
    
    		mobDamage = mob(gen);
    		PlayerHP -= mobDamage;
    		if (PlayerHP < 0) PlayerHP = 0;
    		cout << "몬스터의 공격으로 " << mobDamage << "의 피해를 받았다!" << endl;
    		cout << "당신의 HP: " << PlayerHP << "  /   MP: " << PlayerMP << endl << endl;
    	}
    
    	if (PlayerHP == 0)
    		cout << "===========[ Game Over! ]==========" << endl;
    	else if (MonsterHP == 0)
    		cout << "===========[ Game Clear! ]==========" << endl;

        system("pause");
        
    	return 0;
    }
