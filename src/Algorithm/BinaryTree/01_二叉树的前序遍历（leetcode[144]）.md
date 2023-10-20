# 二叉树的前序遍历

## 1. 递归算法

```java
public List<Integer> preorderTraversal(TreeNode root) {
  List<Integer> result = new ArrayList<>();
  return preOrder(result, root);
}

public List<Integer> preOrder(List<Integer> result, TreeNode root) {
  if (null == root) {
    return result;
  }
  result.add(root.val);
  preOrder(result, root.left);
  preOrder(result, root.right);
  return result;
}
```



## 2. 迭代算法

```java
public List<Integer> preorderTraversal(TreeNode root) {
  List<Integer> list = new ArrayList<>();
  Stack<TreeNode> stack = new Stack<>();
  if (null == root) {
    return list;
  }
  stack.push(root);
  while (!stack.isEmpty()) {
    TreeNode top = stack.pop();
    list.add(top.val);
    if (null != top.left) {
      stack.push(top.left);
    }
    if (null != top.right) {
      stack.push(top.right);
    }
  }
  return list;
}
```

