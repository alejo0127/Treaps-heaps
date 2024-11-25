# Treaps-heaps
Presentacion proyecto final DS2024-2

Carpeta de videos (Si no puede visualizarlo desde el navegador, descargar o usar una extensión de visualización de videos de drive):
https://drive.google.com/drive/folders/1UZGW9SSC7g7ztlgyiUX4DA0EGQ_dwvRG?usp=drive_link

Tablero digital con explicación escrita de las operaciones del Treap:
https://excalidraw.com/#json=THgHssFm7bc6KXbN7SgjC,pQcjlK84pnDFAlx-GbSUlg

En este documento está el código de Treap y MaxHeap

## Código de Treap

```c++
#include <ctime>
#include <iostream>
#include <time.h>
#include <stdlib.h>
#include <random>
#include <chrono>
//Treap's code

using namespace std;
//random number generator
int randNumber() {
    // Seed based on time
    static std::mt19937 rng(std::chrono::steady_clock::now().time_since_epoch().count()); 
    // Numbers range (from 1 to 1000000)
    static std::uniform_int_distribution<int> dist(1, 1000000); 
    return dist(rng);
}
template<typename K>
//Creating class treap
class Treap {
    private:
    //Creating TreapNode class
        class TreapNode {
            public:
            //TreapNode attributes
                K key;
                int priority;
                TreapNode* left;
                TreapNode* right;
            //TreapNode constructor
                TreapNode(K k, int p) : key(k), priority(p), left(nullptr), right(nullptr){}
        };

    //Treap attributes
    TreapNode* root;
    unsigned int size;

    //Rotations
    TreapNode* rotateRight(TreapNode* y) {
        TreapNode* x = y->left;
        y->left = x->right;
        x->right = y;
        return x;
    }
    TreapNode* rotateLeft(TreapNode* y) {
        TreapNode* x = y->right;
        y->right = x->left;
        x->left = y;
        return x;
    }

    //Private insert (The one the insert wrapper calls)

    TreapNode* insert(TreapNode* node, K key) {
        
        //If node does not exist, we insert the new node here 
        if (!node){
            //Generating a new priority and checking if it doesn't exist in the treap already
            int r = randNumber();
            while (1)
            {
               if(!prioritySearch(r)){
                break;
                } 
                r= randNumber();
            }
            size++;
            return new TreapNode(key, r);
        }
        //If the key we want to insert is less than the current node's key, we make a recursive call which parameter is going to be its left, otherwise, we run the else code
        if (key < node->key) {
            node->left = insert(node->left, key);
            //if the current node's left child priority is greater than its own priority, we make a rotation to the right.
            if (node->left->priority > node->priority) {
                node = rotateRight(node);
            }
        } else {
            node->right = insert(node->right, key);
            //if the current node's right child priority is greater than its own priority, we make a rotation to the left.
            if (node->right->priority > node->priority) {
                node = rotateLeft(node);
            }
        }
        return node;
    }

    /*Private insert (The one that search uses when it generates a better priority for the node the user was searching) 
    We use this one because it allows keeping the priority the search function generated previously.*/
    
    TreapNode* insertPriv(TreapNode* node, K key, int r) {
        if (!node){
            size++;
            return new TreapNode(key, r);
        }

        if (key < node->key) {
            node->left = insertPriv(node->left, key, r);
            if (node->left->priority > node->priority) {
                node = rotateRight(node);
            }
        } else {
            node->right = insertPriv(node->right, key, r);
            if (node->right->priority > node->priority) {
                node = rotateLeft(node);
            }
        }
        return node;
    }
    //Private insertPriv wrapper

    void insertPriv2(K key, int r) {
        root = insertPriv(root, key, r);
    }
    //Private deleteNode (The one the deleteNode wrapper calls)

    TreapNode* deleteNode(TreapNode* node, K key) {
        //if node does not exist, return nullptr
        if (!node){
            return nullptr;
        }
        
        if (key < node->key) {
            //if node's key is not less than "key", it's new left is gonna be the recursive's call return
            node->left = deleteNode(node->left, key);
        }
        else if (key > node->key) {
            //if node's key is less than "key", it's new left is gonna be the recursive's call return
            node->right = deleteNode(node->right, key);
        }
        else {
            //if the node's key is the same as key, we first check if it has one, two or no children.
            if (!node->left) {
                TreapNode* temp = node->right;
                delete node;
                size--;
                return temp;
            } else if (!node->right) {
                TreapNode* temp = node->left;
                delete node;
                size--;
                return temp;
            }
            //if it has two children, we check their priorities and if the right child priority is less than the left child priority, then we make a rotation to the right an vice versa.
            if (node->left->priority > node->right->priority) {
                node = rotateRight(node);
                node->right = deleteNode(node->right, key);
            } else {
                node = rotateLeft(node);
                node->left = deleteNode(node->left, key);
            }
        }
        return node;
    }

    TreapNode* searchParent(K key){
        //BST Search
        TreapNode* parent =nullptr;
	    TreapNode* current =root;
	    while(current&&current->key!=key){
            parent=current;
		    if(key<current->key){
			    current=current->left;
		    }
		    else{
			    current=current->right;
		    }
            }
            return parent;
    }

    //We use this function to verify the priority we generated isn't already a priority in one of the Treap nodes.

    bool prioritySearch(int priority){
        //BST search
        TreapNode* current =root;
        while(current&&current->priority!=priority){
            if(priority<current->priority){
                current=current->left;
            }
            else{
                current=current->right;
            }
        }
        if(!current){
            return false;
        }
        return true;
    }
    
    //Private search (The one the search wrapper calls)

    bool search(TreapNode* node, K key) {
        //if node does not exist, we return false
        if (!node) return false;
        //if the key does exist, we run this conditional
        if (node->key == key){
            //we first check if it's the root, because if it is, we have no need to generate a new priority, because it's gonna be the greatest one inside the treap.
            if(!(node==root)){
                //once that we know the node we are searching for isn't the root, we generate a new priority
                int r = randNumber();
                while (1)
                {
                if(!prioritySearch(r)){
                    break;
                    } 
                    r=randNumber();
                }
                K tempKey=node->key;
                //we search the node's parent to check if its priority is less than the priority we just generated.
                TreapNode* parent= searchParent(tempKey);
                if(r>parent->priority){
                    //In case the parent's priority is less than the new priority, we delete the node and reinsert it with the new priority
                    deleteNode(tempKey); 
                    insertPriv2(tempKey, r);
                }
            }
            return true;
            }
            //Depending on if the key we are searching for is less than the current node's key, we make a recursive call which parameter is going to be the right or the left child.
        if (key < node->key) return search(node->left, key);
        return search(node->right, key);
    }

    //The next three functions are for printing purposes

    void inorder(TreapNode* node) const {
        if (node) {
            inorder(node->left);
            printKey(node->key);
            cout<<node->priority<<" ";
            inorder(node->right);
        }
    }

    void printKey(K key) const {
        char buffer[12];
        itoa(key, buffer, 10);
        print(buffer);
        print(":");
        
    }
    void print(const char* str) const {
        while (*str) {
            putchar(*str++);
        }
    }


public:
    Treap() : root(nullptr), size(0) {}

    //Function that returns the size attribute
    unsigned int getSize(){
        return size;
    }
    //Insert wrapper

    void insert(K key) {
        root = insert(root, key);
    }

    //deleteNode wrapper

    void deleteNode(K key) {
        root = deleteNode(root, key);
    }

    //search wrapper

    bool search(K key) {
        return search(root, key);
    }
    
    //inorder wrapper
    void inorder() const {
        inorder(root);
        print("\n");
    }
    
};

int main() {
    Treap<int> treap;
    cout<<"current size (when root is nullptr): ";
    cout<<treap.getSize()<<endl;
    treap.insert(35);
    treap.insert(20);
    treap.insert(5);
    treap.insert(4);

    cout<<"current size: ";
    cout<<treap.getSize()<<endl;

    treap.inorder();
    if(treap.search(5)==true){
        cout<<"found 5"<<endl;
    }
    else{
        cout<<"5 not found"<<endl;
    }

    treap.inorder();
    if(treap.search(0)==true){
        cout<<"found 0"<<endl;
    }
    else{
        cout<<"0 not found"<<endl;
    }
    treap.deleteNode(5);
    treap.insert(2);
    treap.inorder();

    if(treap.search(5)==true){
        cout<<"found 5"<<endl;
    }
    else{
        cout<<"5 not found"<<endl;
    }

    treap.insert(5);
    treap.inorder();
    return 0;
}

```

## Código de MaxHeap

```c++
class MaxHeap {
private:
    vector<int> heap;

    // Returns the index of the parent of the given node
    int getParentIndex(int index) { return (index - 1) / 2; }

    // Returns the index of the left child of the given node
    int getLeftChildIndex(int index) { return 2 * index + 1; }

    // Returns the index of the right child of the given node
    int getRightChildIndex(int index) { return 2 * index + 2; }

    // Checks if the node has a left child
    bool hasLeftChild(int index) { return getLeftChildIndex(index) < heap.size(); }

    // Checks if the node has a right child
    bool hasRightChild(int index) { return getRightChildIndex(index) < heap.size(); }

    // Checks if the node has a parent
    bool hasParent(int index) { return getParentIndex(index) >= 0; }

    // Returns the value of the left child of the given node
    int leftChild(int index) { return heap[getLeftChildIndex(index)]; }

    // Returns the value of the right child of the given node
    int rightChild(int index) { return heap[getRightChildIndex(index)]; }

    // Returns the value of the parent of the given node
    int parent(int index) { return heap[getParentIndex(index)]; }

    // Swaps the values of two nodes in the heap
    void swap(int indexOne, int indexTwo) {
        int temp = heap[indexOne];
        heap[indexOne] = heap[indexTwo];
        heap[indexTwo] = temp;
    }

    // Maintains the max-heap property by moving a node up
    void heapifyUp() {
        int index = heap.size() - 1; // Start with the last inserted node
        while (hasParent(index) && parent(index) < heap[index]) {
            swap(getParentIndex(index), index);
            index = getParentIndex(index); // Move up to the parent node
        }
    }

    // Maintains the max-heap property by moving a node down
    void heapifyDown() {
        int index = 0; // Start with the root
        while (hasLeftChild(index)) { // Continue while there is at least one child
            int largerChildIndex = getLeftChildIndex(index);
            if (hasRightChild(index) && rightChild(index) > leftChild(index)) {
                largerChildIndex = getRightChildIndex(index); // Choose the larger child
            }

            if (heap[index] > heap[largerChildIndex]) {
                break; // Stop if the heap property is satisfied
            } else {
                swap(index, largerChildIndex); // Swap with the larger child
            }
            index = largerChildIndex; // Move down to the larger child
        }
    }

public:
    MaxHeap() {}

    // Inserts a value into the heap
    void insert(int value) {
        heap.push_back(value); // Add the new value at the end
        heapifyUp(); // Restore the heap property
    }

    // Removes and returns the maximum value from the heap
    int remove() {
        if (heap.empty()) {
            throw out_of_range("Heap is empty"); // Handle empty heap case
        }
        int maxValue = heap[0]; // Store the root (max value)
        heap[0] = heap.back(); // Replace root with the last element
        heap.pop_back(); // Remove the last element
        heapifyDown(); // Restore the heap property
        return maxValue;
    }

    // Returns the maximum value without removing it
    int peek() const {
        if (heap.empty()) {
            throw out_of_range("Heap is empty"); // Handle empty heap case
        }
        return heap[0]; // The root is the maximum value
    }

    // Prints the contents of the heap
    void printHeap() const {
        for (int val : heap) {
            cout << val << " "; // Print each value separated by spaces
        }
        cout << endl; // End the line
    }
};

```
