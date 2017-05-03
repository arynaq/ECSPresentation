---
#Entity Component System - A framework
---
#Imagine you are a gamedesigner...
---?image=/inheritance.png
---
## Where does this zombie boar fit?
#Composition to the rescue
## A boar can be a standalone class, maybe
## A boar is composed of objects that define data or logic
---?image=/ecs2.png

#HSLIDE
```c++
class Boar : public Enemy{
    ZombieComponent z;
    ArmorComponent a;
    WeaponComponent w;

    ...
    void onhit(Enemy& other){
        a.durability--;
        w.inflictDamage(other);
    }
}
```

#HSLIDE
#We can do better

#HSLIDE
# Entity Component System
---?image=/ecs2.gif
## Late 90s

#HSLIDE
# Entity
## Identifier
``c++
struct ID {
            using u32 = std::uint32_t;
            u32 index : ENTITY_COUNTER_BITS;   // bitfield 16 bits for this variable
            u32 counter : ENTITY_COUNTER_BITS;

            ID(u32 index, u32 counter) :
                index(index), counter(counter)
            {
            }

            ID() : ID(0,0) {}

            inline
            u32 value() const {
                return (counter << ENTITY_COUNTER_BITS  )| index;
            }
        };
``

#HSLIDE
# Component
## Pure data
```c++
#pragma once
#include "Component.hpp"

struct CollisionComponent : Component {
    float offsetX = 2;
    float offsetY = 2;
    sf::FloatRect collisionBox{offsetX,offsetY,27,30};
};
```
#HSLIDE
# Adding components to entity
```c++
template<typename T, typename... Args>
T& Entity::addComponent(Args&&...args){
    assertComponentDerived<T>();
    auto component = new T{std::forward<Args>(args)...};
    addComponent(component, ComponentTypeID<T>());
    return *component;
}
```

#HSLIDE
# System
## Pure logic, maybe some auxillary data
## Operate on matching entities

#HSLIDE
# Challanges
## Decoupled a lot, but if we need coupling?
## Cache unfriendly (at this point)

#HSLIDE
# Short Demo
## A star AI
The End :)
