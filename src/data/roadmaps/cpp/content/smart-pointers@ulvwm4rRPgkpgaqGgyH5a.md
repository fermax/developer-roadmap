# Smart Pointers
Smart pointers are objects that act like pointers but provide automatic memory management. They help prevent memory leaks by automatically deallocating memory when the pointer goes out of scope. C++ offers several types of smart pointers in the <memory> header.

Types of Smart Pointers
1. unique_ptr
Exclusive ownership of the object

Can't be copied (only moved)

Lightweight, zero overhead

2. shared_ptr
Shared ownership (reference counting)

Multiple pointers can point to the same object

Slightly heavier due to reference counting

3. weak_ptr
Non-owning observer of a shared_ptr

Doesn't increase reference count

Used to break circular references

Examples
Basic Usage


#include <memory>
#include <iostream>

class MyClass {
public:
    MyClass() { std::cout << "MyClass constructed\n"; }
    ~MyClass() { std::cout << "MyClass destroyed\n"; }
    void greet() { std::cout << "Hello from MyClass!\n"; }
};

void basicExample() {
    std::unique_ptr<MyClass> ptr1(new MyClass());
    ptr1->greet();
    
    // Automatically deleted when ptr1 goes out of scope
}

unique_ptr Examples

void uniquePtrExamples() {
    // Creation
    std::unique_ptr<int> u1(new int(10));
    std::unique_ptr<int> u2 = std::make_unique<int>(20); // Preferred (C++14)
    
    // Move semantics (ownership transfer)
    std::unique_ptr<int> u3 = std::move(u1); // u1 is now nullptr
    
    // Array support
    std::unique_ptr<int[]> arr(new int[5]{1,2,3,4,5});
    arr[2] = 10;
    
    // Custom deleter
    auto deleter = [](int* p) { 
        std::cout << "Custom delete\n"; 
        delete p; 
    };
    std::unique_ptr<int, decltype(deleter)> u4(new int(30), deleter);
    
    // Release ownership
    int* rawPtr = u2.release();
    delete rawPtr;
}

shared_ptr Examples

void sharedPtrExamples() {
    // Creation
    std::shared_ptr<MyClass> s1(new MyClass());
    std::shared_ptr<MyClass> s2 = std::make_shared<MyClass>(); // Preferred
    
    // Copying increases reference count
    std::shared_ptr<MyClass> s3 = s2;
    std::cout << "Use count: " << s2.use_count() << "\n"; // 2
    
    // Arrays (since C++17)
    std::shared_ptr<int[]> arr(new int[5]{1,2,3,4,5});
    arr[2] = 10;
    
    // Custom deleter
    std::shared_ptr<int> s4(new int, [](int* p) { 
        std::cout << "Custom delete\n"; 
        delete p; 
    });
    
    // Aliasing constructor (sharing ownership but pointing to different object)
    struct Parent { int data; };
    struct Child { };
    
    std::shared_ptr<Parent> parent(new Parent{42});
    std::shared_ptr<Child> child(parent, &parent->child);
}

weak_ptr Examples

void weakPtrExamples() {
    std::shared_ptr<MyClass> shared = std::make_shared<MyClass>();
    std::weak_ptr<MyClass> weak = shared;
    
    // Check if the object still exists
    if (auto temp = weak.lock()) {
        temp->greet();
        std::cout << "Use count: " << temp.use_count() << "\n";
    } else {
        std::cout << "Object already destroyed\n";
    }
    
    // Breaking circular references
    struct Node {
        std::shared_ptr<Node> next;
        std::weak_ptr<Node> prev; // Use weak_ptr to break cycle
        ~Node() { std::cout << "Node destroyed\n"; }
    };
    
    auto node1 = std::make_shared<Node>();
    auto node2 = std::make_shared<Node>();
    node1->next = node2;
    node2->prev = node1;
}
