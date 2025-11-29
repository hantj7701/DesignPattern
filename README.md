전략 패턴과 팩토리 메서드를 합친 코드에 추가 헤더 파일과 함께 객체 어댑터 패턴을 적용하여 구현 (Adapter_FactoryMethod_Strategy.cpp)

    #include <iostream>
    #include <memory>
    #include "Hammer.h"
    #include "Katana.h"
    
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
    
    // Adapter: KatanaAdapter
    class KatanaAdapter : public Weapon {
        unique_ptr<Katana> katana;
    
        class KatanaAttackStrategy : public AttackStrategy {
            Katana* kat;
        public:
            explicit KatanaAttackStrategy(Katana* k) : kat(k) {}
            void attack(int damage) override {
                cout << "Katana의 slash로 " << damage << "의 피해를 입혔다! -> ";
                kat->slash();
            }
        };
    
    public:
        KatanaAdapter() : katana(make_unique<Katana>()) {
            SetAttackStrategy(new KatanaAttackStrategy(katana.get()));
        }
    
        void equip() override { cout << "Katana 장착" << endl; }
    
        ~KatanaAdapter() override = default;
    };
    
    // Adapter: HammerAdapter
    class HammerAdapter : public Weapon {
        unique_ptr<Hammer> hammer;
    
        class HammerAttackStrategy : public AttackStrategy {
            Hammer* h;
        public:
            explicit HammerAttackStrategy(Hammer* hh) : h(hh) {}
            void attack(int damage) override {
                cout << "Hammer의 smash로 " << damage << "의 피해를 입혔다! -> ";
                h->smash();
            }
        };
    
    public:
        HammerAdapter() : hammer(make_unique<Hammer>()) {
            SetAttackStrategy(new HammerAttackStrategy(hammer.get()));
        }
    
        void equip() override { cout << "Hammer 장착" << endl; }
    
        ~HammerAdapter() override = default;
    };
    
    // Adapter 팩토리들 (Adapter를 팩토리와 연결)
    class KatanaFactory : public WeaponFactory {
    public:
        unique_ptr<Weapon> CreateWeapon() override {
            return make_unique<KatanaAdapter>();
        }
    };
    
    class HammerFactory : public WeaponFactory {
    public:
        unique_ptr<Weapon> CreateWeapon() override {
            return make_unique<HammerAdapter>();
        }
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
        cout << endl;
    
        // 외부 무기 Katana
        KatanaFactory kf;
        auto katanaWeapon = kf.CreateWeapon();
        katanaWeapon->equip();
        katanaWeapon->AttackDamage(30);
    
        // 전략 교체 -> 내부 전략
        katanaWeapon->SetAttackStrategy(new NormalAttack());
        katanaWeapon->AttackDamage(35);
        katanaWeapon->SetAttackStrategy(new ChargedAttack());
        katanaWeapon->AttackDamage(45);
        katanaWeapon->SetAttackStrategy(new ComboAttack());
        katanaWeapon->AttackDamage(80);
        cout << endl;
    
        // 외부 무기 Hammer
        HammerFactory hf;
        auto hammerWeapon = hf.CreateWeapon();
        hammerWeapon->equip();
        hammerWeapon->AttackDamage(200);
    
        // 전략 교체 -> 내부 전략
        hammerWeapon->SetAttackStrategy(new NormalAttack());
        hammerWeapon->AttackDamage(40);
        hammerWeapon->SetAttackStrategy(new ChargedAttack());
        hammerWeapon->AttackDamage(60);
        hammerWeapon->SetAttackStrategy(new ComboAttack());
        hammerWeapon->AttackDamage(100);
    
    
        return 0;
    
    }


객체 어댑터로 적용시킬 외부 헤더 파일(1) (Hammer.h)

    #pragma once
    #include <iostream>
    using namespace std;
    
    class Hammer {
    public:
        void smash() {
            cout << "Hammer smash!!" << endl;
        }
    };


객체 어댑터로 적용시킬 외부 헤더 파일(2) (Katana.h)

    #pragma once
    #include <iostream>
    using namespace std;
    
    class Katana {
    public:
        void slash() {
            cout << "Katana slash!!" << endl;
        }
    };
