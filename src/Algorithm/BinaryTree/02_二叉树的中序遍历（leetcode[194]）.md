# 二叉树的中序遍历

## 1. 递归算法

```java
public List<Integer> inorderTraversal(TreeNode root) {
  if(root != null) {
    inorderTraversal(root.left);
    list.add(root.val);
    inorderTraversal(root.right);
  }
  return list;
}
```



## 2. 迭代算法

```java
List<Integer> list = new ArrayList<>();
Stack<TreeNode> stack = new Stack<>();

public List<Integer> inorderTraversal(TreeNode root) {
  while (root != null || !stack.isEmpty()) {
    while (root != null) {
      stack.push(root);
      root = root.left;
    }
    root = stack.pop();
    list.add(root.val);
    root = root.right;
  }
  return list;
}
```

