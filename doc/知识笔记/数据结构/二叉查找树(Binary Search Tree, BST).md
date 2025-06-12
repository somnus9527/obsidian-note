---
tags:
  - 知识笔记/数据结构
---
二叉查找树（BST：Binary Search Tree）是一种特殊的二叉树，它改善了二叉树节点查找的效率。二叉查找树有以下性质：  
对于任意一个节点 n，

- 其左子树（left subtree）下的每个后代节点（descendant node）的值都小于节点 n 的值；
- 其右子树（right subtree）下的每个后代节点的值都大于节点 n 的值。

1. 搜索 查找目标值,小于当前节点就往左走,大于就往右走
2. 插入 先搜索,找到该值的父节点,逻辑就是搜索的逻辑
3. 删除

- 情况 1：如果删除的节点没有右孩子，那么就选择它的左孩子来代替原来的节点。二叉查找树的性质保证了被删除节点的左子树必然符合二叉查找树的性质。因此左子树的值要么都大于，要么都小于被删除节点的父节点的值，这取决于被删除节点是左孩子还是右孩子。因此用被删除节点的左子树来替代被删除节点，是完全符合二叉搜索树的性质的。
- 情况 2：如果被删除节点的右孩子没有左孩子，那么这个右孩子被用来替换被删除节点。因为被删除节点的右孩子都大于被删除节点左子树的所有节点，同时也大于或小于被删除节点的父节点，这同样取决于被删除节点是左孩子还是右孩子。因此，用右孩子来替换被删除节点，符合二叉查找树的性质。
- 情况 3：如果被删除节点的右孩子有左孩子，就需要用被删除节点右孩子的左子树中的最下面的节点来替换它，就是说，我们用被删除节点的右子树中最小值的节点来替换。
![[Pasted image 20241231185036.png]]![[Pasted image 20241231185047.png]]
>**BST复杂度都是 最佳情况是 O(log­2n)，而最坏情况是 O(n)**

```typescript
import { TreeNode } from '@/data_structure/BinaryTree/BinaryTree';
import { Stack } from '@/data_structure/Stack/Stack';

// 二叉搜索树,不允许重复值
class BSTree<T extends string|number> {
    private _root: TreeNode<T> = null;

    constructor(root?: TreeNode<T>) {
        if(root)
            this._root = root ?? null;
    }

    _search(target: T,root: TreeNode<T>): TreeNode<T> {
        if(root === null) return null;
        let current: TreeNode<T> = root;
        while (current.data !== target) {
            if(target > current.data) {
                if(current.right) current = current.right;
                return current;
            } else if(target < current.data) {
                if(current.left) current = current.left;
                return current;
            }
        }
        return current;
    }

    search(target: T): TreeNode<T> {
        const node: TreeNode<T> = this._search(target,this._root);
        if(node === null || node.data !== target) return null;
        return node;
    }

    insert(node: TreeNode<T>): TreeNode<T> {
        const value: T = node.data;
        const target: TreeNode<T> = this._search(value,this._root);
        if(target === null) return this._root;
        if(target.data > node.data) target.left = node;
        if(target.data < node.data) target.right = node;
        return this._root;
    }

    _searchParent(target: T,root: TreeNode<T>,rootParent: TreeNode<T> = null): [TreeNode<T>,TreeNode<T>] {
        let current: TreeNode<T> = root;
        let currentParent: TreeNode<T> = rootParent;
        while (current.data !== target) {
            if(target > current.data) {
                if(current.right) {
                    currentParent = current;
                    current = current.right;
                }
                return [current,currentParent];
            } else if(target < current.data) {
                if(current.left) {
                    currentParent = current;
                    current = current.left;
                }
                return [current,currentParent];
            }
        }
        return [current,currentParent];
    }

    _leftOrRight(pnode: TreeNode<T>,data: T): string {
        if(pnode === null) return null;
        if(pnode.left !== null && pnode.left.data === data) return 'left';
        return 'right';
    }

    // 中序遍历
    _midSearch(root: TreeNode<T>): TreeNode<T>[] {
        if(root === null) return null;
        const cacheVisit: TreeNode<T>[] = [root];
        const stack: Stack<TreeNode<T>> = new Stack<TreeNode<T>>();
        const result: TreeNode<T>[] = [];
        stack.push(root.right);
        stack.push(root);
        stack.push(root.left);
        while(!stack.empty) {
            const node: TreeNode<T> = stack.pop();
            cacheVisit.push(node);
            if(( node.left === null && node.right === null ) || cacheVisit.includes(node)) {
                result.push(node);
            } else {
                if(node.right !== null) stack.push(node.right);
                stack.push(node);
                if(node.left !== null) stack.push(node.left);
            }
        }
        return result;
    }

    delete(target: TreeNode<T>): TreeNode<T> {
        if(this._root === null) return null;
        const value: T = target.data;
        const [pnode,dnode] = this._searchParent(value,this._root,null);
        if(dnode === null) return this._root;
        if(dnode.left === null && dnode.right === null) {
            if(pnode === null) this._root = null;
            else {
                const direction: string = this._leftOrRight(pnode,dnode.data);
                pnode[direction] = null;
            }
        } else if(dnode.left !== null && dnode.right !== null) {
            const midResult: TreeNode<T>[] = this._midSearch(dnode.right);
            const rnode: TreeNode<T> = midResult[0];
            const prnode: TreeNode<T> = midResult[1];
            const rDirection: string = this._leftOrRight(prnode,rnode.data);
            prnode[rDirection] = rnode.right;
            rnode.left = dnode.left;
            rnode.right = dnode.right;
            if(pnode !== null) {
                const direction: string = this._leftOrRight(pnode,dnode.data);
                pnode[direction] = rnode;
            } else this._root = rnode;
        } else {
            if(pnode === null) this._root = dnode.left ?? dnode.right;
            else {
                const direction: string = this._leftOrRight(pnode,dnode.data);
                pnode[direction] = dnode.left ?? dnode.right;
            }
        }
    }

}

export {
    BSTree,
}
```