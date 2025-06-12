---
tags:
  - 知识笔记/数据结构
---
>**在BST的基础上,AVL树要求，任一节点对应的两棵子树的最大高度差为1，因此它也被称为高度平衡树。**

```typescript
enum TreeNodeDataEqualType {
    EQUAL,
    BIGGER,
    LESS,
}

abstract class TreeNodeData<T> {
    data: T;

    constructor(data: T) {
        this.data = data;
    }

    abstract isEqual(data: T): TreeNodeDataEqualType;
}

class Data extends TreeNodeData<number> {
    isEqual(data: number): TreeNodeDataEqualType {
        if(this.data === data) return TreeNodeDataEqualType.EQUAL;
        if(this.data > data) return TreeNodeDataEqualType.BIGGER;
        return TreeNodeDataEqualType.LESS;
    }
}

class TreeNode<T extends TreeNodeData<T>> {
    data: T;
    height: number;
    left: TreeNode<T>|null;
    right: TreeNode<T>|null;

    // 默认高度是1,是为了将0留给节点为null的情况
    constructor({data,height = 1,left = null,right = null}: {data: T,height?: number,left?: TreeNode<T>,right?: TreeNode<T>}) {
        this.data = data;
        this.height = height;
        this.left = left;
        this.right = right;
    }

    compare(data): TreeNodeDataEqualType {
        return this.data.isEqual(data.data);
    }
}

// 在AVL树中，任一节点对应的两棵子树的最大高度差为1，因此它也被称为高度平衡树。查找、插入和删除在平均和最坏情况下的时间复杂度都是O(logn)。增加和删除元素的操作则可能需要借由一次或多次树旋转，以实现树的重新平衡。
// 实现不允许出现相等的值
class AVLTree<T extends TreeNodeData<T>> {
    root: TreeNode<T> = null;

    private _maxInt(data1: number,data2: number): number {
        return data1 > data2 ? data1 : data2;
    }

    // 这里注意节点刚刚创建时,高度默认为1,null节点高度认为是0
    getHeight(node: TreeNode<T>): number {
        if(node === null) return 0;
        return node.height;
    }

    // 左右子树的高度差是为这里的balance,大于1|小于-1是为非平衡,这里是左-右,所有大于0是左子树高,小于0是右子树高
    getBalance(node: TreeNode<T>): number {
        if(node === null) return 0;
        return this.getHeight(node.left) - this.getHeight(node.right);
    }

    // 左旋: gif地址 https://img-blog.csdn.net/20180722220546910
    private _leftRotate(node: TreeNode<T>): TreeNode<T> {
        const rightNode: TreeNode<T> = node.right;
        const rlNode: TreeNode<T> = rightNode.left;
        // 开始左旋
        node.right = rlNode;
        rightNode.left = node;
        // 更新高度,这里可以画图看一下,左旋只会影响下面两个节点的高度,要注意的是先更新node节点后更新rightNode节点
        node.height = this._maxInt(this.getHeight(node.left),this.getHeight(node.right)) + 1;
        rightNode.height = this._maxInt(this.getHeight(rightNode.left),this.getHeight(rightNode.right)) + 1;

        return rightNode;
    }

    // 右旋: gif地址 https://img-blog.csdn.net/20180722222413303
    private _rightRotate(node: TreeNode<T>): TreeNode<T> {
        const leftNode: TreeNode<T> = node.left;
        const lrNode: TreeNode<T> = leftNode.right;
        // 开始右旋
        node.left = lrNode;
        leftNode.right = node;
        // 更新高度
        node.height = this._maxInt(this.getHeight(node.left),this.getHeight(node.right)) + 1;
        leftNode.height = this._maxInt(this.getHeight(leftNode.left),this.getHeight(leftNode.right)) + 1;
        return leftNode;
    }

    // 插入方法
    private _insert(node: TreeNode<T>|null,data: T): TreeNode<T> {
        if (node === null) return new TreeNode<T>({data});
        // 插入值小于节点值,往左走
        if (node.compare(data) === TreeNodeDataEqualType.BIGGER)
            node.left = this._insert(node.left, data);
        // 插入值大于节点值,往右走
        else if (node.compare(data) === TreeNodeDataEqualType.LESS)
            node.right = this._insert(node.right, data);
        // 不允许相同的值
        else return node;
        // 这里开始递归从下往上开始走
        // 首先处理一下每个节点的高度
        node.height = 1 + this._maxInt(this.getHeight(node.left), this.getHeight(node.right));
        // 获取balance,开始判断所有节点是否平衡
        const balance: number = this.getBalance(node);
        // 四个if分支的操作,前提条件是不平衡,平衡是不走的,不平衡的条件是 balance > 1 (左子树更高) || balance < -1 (右子树更高);
        // 情况1: 左左,条件是: 1. 左子树更高 2.插入的数据比当前节点左子树数据还要小
        if (balance > 1 && data.isEqual(node.left.data.data) === TreeNodeDataEqualType.LESS)
            return this._rightRotate(node);
        // 情况2: 右右,条件是: 1. 右子树更高 2.插入数据比当前节点右子树数据还要大
        if(balance < -1 && data.isEqual(node.right.data.data) === TreeNodeDataEqualType.BIGGER)
            return this._leftRotate(node);
        // 情况3: 左右,条件是: 1. 左子树更高 2.插入数据比当前节点左子树数据要大
        if(balance > 1 && data.isEqual(node.left.data.data) === TreeNodeDataEqualType.BIGGER) {
            node.left = this._leftRotate(node.left);
            return this._rightRotate(node);
        }
        // 情况4: 右左,条件是: 1. 右子树更高 2.插入数据比当前节点右子树数据要小
        if(balance < -1 && data.isEqual(node.right.data.data) === TreeNodeDataEqualType.LESS) {
            node.right = this._rightRotate(node.right);
            return this._leftRotate(node);
        }
        // 如果是平衡的,则正常返回原节点
        return node;
    }

    insert(data: T) {
        if(data === null) return this.root;
        this.root = this._insert(this.root,data);
        return this.root;
    }

    private _minValueNode(node: TreeNode<T>): TreeNode<T> {
        let current: TreeNode<T> = node;
        while (current.left !== null) {
            current = current.left;
        }
        return current;
    }

    // 删除方法
    private _delete(node: TreeNode<T>|null,data: T): TreeNode<T> {
        // 节点为空,说明已经找到了根节点还是没有找到,直接返回原节点即可
        if (node === null) return node;
        // 需要删除的值小于节点值,继续往左分支递归删除
        if (data.isEqual(node.data.data) === TreeNodeDataEqualType.LESS)
            node.left = this._delete(node.left, data);
        // 需要删除的值大于节点值,继续往右分支递归删除
        else if (data.isEqual(node.data.data) === TreeNodeDataEqualType.BIGGER)
            node.right = this._delete(node.right, data);
        else {
            // 进入这里就说明找到需要删除的节点了
            // 这个分支是判断需要删除的节点是否左右分支是否有一个为null
            if ((node.left === null) || (node.right === null)) {
                let temp: TreeNode<T> | null = null;
                if (temp === node.left)
                    temp = node.right;
                else
                    temp = node.left;
                // 这段记录任意一个部位空的分支到temp
                // 如果temp为null说明左右子节点都为null那么把node置空
                if (temp === null)
                    node = null;
                // 如果temp有值,直接把这个值赋值给node节点
                else
                    node = temp;
            } else {
                // 如果node节点左右分支都不为空,那么找到node右子树的最小值节点,并记录到temp
                let temp: TreeNode<T> | null = this._minValueNode(node.right);
                // 修改node的值为temp的值
                node.data = temp.data;
                // 继续递归删除node节点的右子树,找的值是temp的值,所以上一步已经记录值到node节点了
                node.right = this._delete(node.right, temp.data);
            }
        }
        // 到这节点增删已经处理完成了,后面要做的事就是rebalance了,这块跟insert是一样的
        // 这里说明整棵树只有一个节点
        if (node === null) return node;
        // 更新高度
        node.height = 1 + this._maxInt(this.getHeight(node.left), this.getHeight(node.right));
        // 获取balance,准备旋转
        const balance: number = this.getBalance(node);
        // 情况1: 左左
        if (balance > 1 && this.getBalance(node.left) >= 0)
            return this._rightRotate(node);
        // 情况2: 左右
        if (balance > 1 && this.getBalance(node.left) < 0) {
            node.left = this._leftRotate(node.left);
            return this._rightRotate(node);
        }
        // 情况3: 右右
        if (balance < -1 && this.getBalance(node.right) <= 0)
            return this._leftRotate(node);
        // 情况4: 右左
        if (balance < -1 && this.getBalance(node.right) > 0) {
            node.right = this._rightRotate(node.right);
            return this._leftRotate(node);
        }

        return node;
    }

    delete(data: T) {
        if(data === null) return this.root;
        this.root = this._delete(this.root,data);
        return this.root;
    }

    preOrder(node: TreeNode<T>) {
        if (node != null) {
            console.log(node.data.data + " ");
            this.preOrder(node.left);
            this.preOrder(node.right);
        }
    }
}

export {
    AVLTree,
    TreeNode,
    TreeNodeData,
    TreeNodeDataEqualType,
    Data,
}
```