#HSLIDE
#A framework..

#HSLIDE
#Imagine you are a gamedesigner...
---?image=/inheritance.png

#HSLIDE
## Where does this zombie boar fit?

#HSLIDE
##Composition to the rescue
### A boar can be a standalone class, maybe
### A boar is composed of objects that define data or logic
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
```c++
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
```

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
### Adding components to entity
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
### Pure logic, maybe some auxillary data

### Operate on matching entities

#HSLIDE
# Challanges
#HSLIDE
#Decoupled a lot, but if we need coupling?
#HSLIDE
#Cache unfriendly (at this point)

#HSLIDE
### Event Manager
```c++
using Signal = boost::signals2::signal<void(const void*)>;
template <typename E, typename Subscriber>
    void subscribe(Subscriber& subscriber){
        /** Cannot use auto for the callback, as overloaded functions will fudge up... **/
        static_assert(std::is_base_of<Receiver<Subscriber>, Subscriber>::value, "Subscriber must derive from receiver!..");
        void (Subscriber::*callback)(const E&) = &Subscriber::receive;
        auto callbackWrapper = CallbackWrapper<E>(boost::bind(callback, &subscriber, _1));
        TypeID id = Event<E>::getID();  
        auto connection = map[id].connect(callbackWrapper);
        subscriber.connections[id]= connection;
    }
```
#HSLIDE
### Callback
```c++
template <typename E>
    struct CallbackWrapper {
        explicit CallbackWrapper(boost::function<void(const E&)> callback) :callback(callback)
        {
        }
        void operator()(const void* event){
            callback(*(static_cast<const E*>(event)));
        }
        boost::function<void(const E&)> callback;
    };
```
#HSLIDE 
```c++
struct CollisionResolutionEvent{
    sf::Vector2f moveBy;
    Entity entity;
    CollisionResolutionEvent(sf::Vector2f moveBy, Entity entity):
        moveBy(moveBy),entity(entity)
    {
    }
};
```
#HSLIDE
### Example
```c++
getWorld().messageHandler().emit<CollisionResolutionEvent>(dr, entity);
....
void MovementSystem::receive(const CollisionResolutionEvent& event){
    if(event.entity.hasComponent<TransformComponent>()){
        auto& transform = event.entity.getComponent<TransformComponent>().transform;
        transform.move(event.moveBy);
    }
}
```

#HSLIDE
# Short Demo
#HSLIDE 
# A star AI
The End :)
