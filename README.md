전략 패턴 구현 (Strategy.cpp)

    #include <iostream>
    #include <memory>
    using namespace std;
    
    // 공격 전략 인터페이스 (Strategy)
    class AttackStrategy {
    public:
        virtual void attack(int damage) = 0;
        virtual ~AttackStrategy() {}
    };
    
    // 아래는 구체적인 전략들 (ConcreteStrategy)
    // 일반 공격
    class NormalAttack : public AttackStrategy {
    public:
        void attack(int damage) override {
            cout << "일반 공격으로 " << damage << "의 피해을 입혔다!" << endl;
        }
    };
    
    // 강공격
    class ChargedAttack : public AttackStrategy {
    public:
        void attack(int damage) override {
            cout << "강공격으로 " << damage << "의 피해을 입혔다!" << endl;
        }
    };
    
    // 연속 공격
    class ComboAttack : public AttackStrategy {
    public:
        void attack(int damage) override {
            cout << "연속 공격으로 " << damage << "의 피해을 입혔다!" << endl;
        }
    };
    
    // 공격 전략 사용 주체 (Context)
    class Player {
        unique_ptr<AttackStrategy> strategy;
    
    public:
        void SetAttackStrategy(AttackStrategy* newStrategy) {
            strategy.reset(newStrategy);
        }
    
        void AttackDamage(int damage) {
            if (strategy)
                strategy->attack(damage);
            else
                cout << "아직 공격 전략을 선택하지 못했다..." << endl;
        }
    };
    
    int main() {
        Player p1;
    
        // 공격 전략 선택 X
        p1.AttackDamage(10);
    
        // 일반 공격 전략 선택
        p1.SetAttackStrategy(new NormalAttack());
        p1.AttackDamage(10);
    
        // 강공격 전략 선택
        p1.SetAttackStrategy(new ChargedAttack());
        p1.AttackDamage(20);
    
        // 연속 공격 전략 선택
        p1.SetAttackStrategy(new ComboAttack());
        p1.AttackDamage(40);
    }
