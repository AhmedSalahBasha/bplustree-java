<h1>Database Technology - Programming Exercise: B+ Tree</h1>
The task in this programming exercise is to implement a B+ tree. This document describes
the code template, gives some hints on the implementation and how to use test cases to
drive the implementation. 

<h3>Code template</h3>
The implementation consists of four classes: BPlusTree, Node, InnerNode, and LeafNode.
Details of the classes are given below. The code template requires Java 8 and also works
with Java 11.

<h4>Node class</h4>
This abstract class is the base class for inner nodes and leaf nodes. It stores an array of
Integer keys and has abstract accessor methods for the node’s payload. The payload can be
the values stored in the B+ tree (leaf nodes) or other nodes (inner nodes).</br>
<pre>public abstract class Node {
protected Integer[] keys;
public Node(Integer[] keys, int capacity) { … }
public Integer[] getKeys() { … }
public void setKeys(Integer[] keys) { … }
public abstract Object[] getPayload();
public abstract void setPayload(Object[] payload);
}</pre>
Note that the class enforces that keys.length == capacity.
<h4>InnerNode class</h4>
This class implements the inner nodes of the B+ tree. It extends the Node class and stores
an array of children, i.e., references to other nodes.
<pre>public class InnerNode extends Node {
private Node[] children;
public InnerNode(Integer[] keys, Node[] children, int capacity) { … }
public Node[] getChildren() { … }
public void setChildren(Node[] children) { … }
…
}</pre>
Note that the class enforces that children.length == capacity+1 == keys.length+1.
<h4>LeafNode class</h4>
This class implements the leaf nodes of the B+ tree, i.e., the storage of values. It also
extends the Node class and stores an array of Strings as values.
<pre>public class LeafNode extends Node {
private String[] values;
public LeafNode(Integer[] keys, String[] values, int capacity) { … }
public String[] getValues() { … }
public void setValues(String[] values) { … }
…
}</pre>
Note that the class enforces that values.length == capacity == keys.length. The
class does not have a pointer to the next leaf. We omit this requirement for this exercise.
<h4>BPlusTree class</h4>
This class implements the B+ tree. It contains public methods to lookup a values by a key,
insert a key/value pair, and delete a value by a key.
<pre>public class BPlusTree {
private int capacity;
private Node root;
public BPlusTree(Node root, int capacity) { … }
public String lookup(Integer key) { … }
public void insert(int key, String value) { … }
public String delete(Integer key) { … }
...
}</pre>
Note that the class enforces that capacity is an even number. However, it is your
responsibility to enforce the B+ tree conditions:</br>
● The keys in each node are sorted.
● Each node (except the root) should contain at least capacity/2 keys.
● For each innerNode, the following conditions must hold:
Given:
<pre>
Integer[] keys = innerNode.getKeys();
Node[] children = innerNode.getChildren();
</pre>
Then:
<ul>
<li>If keys[i] == null then children[i+1] == null.</li>
<li>All keys in children[i].getKeys() are smaller than keys[i].</li>
<li>All keys in children[j].getKeys() are greater or equal to keys[i] if j > i.</li>
</ul>
<h4>Helper classes</h4>
In addition, there are a two helper classes to create a String representation of a B+ tree
(BPlusTreePrinter) and to create trees, inner and leaf nodes, and key and value arrays
(BPlusTreeUtilities).
<h3>Implementation hints</h3>
Each of the three operations lookup, insertion, and deletion can be broken down into three
steps:
<ol>
<li>Traverse the B+ tree to find the leaf that should contain the key.</li>
<li>Manipulate the leaf (lookup the value, insert the key/value pair, or delete the key/value pair).</li>
<li>Propagate changes up the tree if necessary.</li>
</ol>
It is useful to track parent nodes while traversing the tree in step one because we might have
to propagate changes up the tree. Because the basic pattern is the same for each public
operation, the public methods described above are already implemented. In addition, the
BPlusTree class contains the following private helper methods that encapsulate these steps.
<pre>
private LeafNode findLeafNode(Integer key, Node node,
Deque<InnerNode> parents) {
if (node instanceof LeafNode) { return (LeafNode) node; }
else {
InnerNode innerNode = (InnerNode) node;
if (parents != null) { parents.push(innerNode); }
// TODO: traverse inner nodes to find leaf node
}
}
private String lookupInLeafNode(Integer key, LeafNode node) {
// TODO: lookup value in leaf node
}
private void insertIntoLeafNode(Integer key, String value,
LeafNode node, Deque<InnerNode> parents) {
// TODO: insert value into leaf node (and propagate changes up)
}
private String deleteFromLeafNode(Integer key, LeafNode node,
Deque<InnerNode> parents) {
// TODO: delete value from leaf node (and propagate changes up)
}
</pre>
Implementing these methods correctly is sufficient to pass the exercise. Note that it is
possible to contain all additional code in the BPlusTree class. However, you can also add
code to the Node, LeafNode, and InnerNode classes if you wish. You should not change
existing code.
</br>
<h3>Stealing and merging during deletion</h3>
If a node does not have enough keys after deletion, it should first steal from its left sibling. If
that is not possible, it should steal from its right sibling. If that is not possible, it should merge
with its right sibling. Finally, if that is not possible it should merge with its left sibling.
<h4>Test cases</h4>
The code template includes a test class (BPlusTreeTest) containing unit tests that help with
the implementation of the B+ tree. The evaluation server performs many more tests and
covers many corner cases. Therefore, passing all unit tests in the code template does
not guarantee that you get all points in the exercise. It should, however, allow you to
write additional tests for corner cases. The unit tests all follow the following template:
<pre>
@Test
public void deleteFromLeaf() {
// given
tree = newTree(newLeaf(keys(1, 2, 3), values("a", "b", "c")));
// when
String value = tree.delete(2);
// then
assertThat(value, is("b"));
assertThat(tree, isTree(
newTree(newLeaf(keys(1, 3), values("a", "c")))));
}
</pre>
First, a new tree is created in the given section. The tree has a default capacity of 4. This
section uses the helper methods newTree, newLeaf, newNode, keys, values, and nodes
from the class BPlusTreeUtilities.
Then, the tree is manipulated in the when section. In this case, a key is deleted and the
corresponding value (if any) is stored in a variable.</br>
Finally, the resulting tree is compared against a reference tree in the then section. The
reference tree is constructed in the same way as the original tree in the given section. It is
then passed to a JUnit matcher using the isTree method and verified using a JUnit
assertion. Because this test case tests for the deletion, the returned value is also verified.
