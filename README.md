구현한 전략 패턴을 합치며 팩토리 메서드 패턴 구현 (FactoryMethod_Strategy.cpp)
    #include <iostream>
    #include <memory>
    
    using namespace std;
    
    // Strategy
    class AttackStrategy {
    public:
        virtual void attack(int damage) = 0;
        virtual ~AttackStrategy() {}
    };
    
    // Product
    class Weapon {
        unique_ptr<AttackStrategy> strategy;
    
    public:
        virtual void equip() = 0;
    
        void SetAttackStrategy(AttackStrategy* newStrategy) {
            strategy.reset(newStrategy);
        }
    
        void AttackDamage(int damage) {
            if (strategy)
                strategy->attack(damage);
            else
                cout << "아직 공격 전략을 선택하지 못했다..." << endl;
        }
    
        virtual ~Weapon() {}
    };
    
    // ConcreteStrategy
    class NormalAttack : public AttackStrategy {
    public:
        void attack(int damage) override {
            cout << "일반 공격으로 " << damage << "의 피해를 입혔다!" << endl;
        }
    };
    
    class ChargedAttack : public AttackStrategy {
    public:
        void attack(int damage) override {
            cout << "강공격으로 " << damage << "의 피해를 입혔다!" << endl;
        }
    };
    
    class ComboAttack : public AttackStrategy {
    public:
        void attack(int damage) override {
            cout << "연속 공격으로 " << damage << "의 피해를 입혔다!" << endl;
        }
    };
    
    // ConcreteProduct
    class Sword : public Weapon {
    public:
        void equip() override { cout << "Sword 장착" << endl; }
    };
    
    class Bow : public Weapon {
    public:
        void equip() override { cout << "Bow 장착" << endl; }
    };
    
    class Gun : public Weapon {
    public:
        void equip() override { cout << "Gun 장착" << endl; }
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
        unique_ptr<Weapon> CreateWeapon() override { return make_unique<Sword>(); }
    };
    
    class BowFactory : public WeaponFactory {
    public:
        unique_ptr<Weapon> CreateWeapon() override { return make_unique<Bow>(); }
    };
    
    class GunFactory : public WeaponFactory {
    public:
        unique_ptr<Weapon> CreateWeapon() override { return make_unique<Gun>(); }
    };
    
    int main()
    {
        SwordFactory SF;
    
        auto sword = SF.CreateWeapon();
    
        sword->equip();
        sword->SetAttackStrategy(new NormalAttack());
        sword->AttackDamage(20);
        sword->SetAttackStrategy(new ChargedAttack());
        sword->AttackDamage(40);
        sword->SetAttackStrategy(new ComboAttack());
        sword->AttackDamage(60);
    }
