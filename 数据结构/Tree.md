#Tree
##二叉树
二叉树维护一个根节点
###数据结构
```
//二叉树的数据储存结构
public class BinaryNode<T extends Comparable> implements Serializable{
    private static final long serialVersionUID = -6477238039299912313L;
    public BinaryNode<T> left;//左结点
    public BinaryNode<T> right;//右结点
    public T data;
    public BinaryNode(T data,BinaryNode left,BinaryNode right){
        this.data=data;
        this.left=left;
        this.right=right;
    }
    public BinaryNode(T data){
        this(data,null,null);
    }
    /**
     * 判断是否为叶子结点
     * @return
     */
    public boolean isLeaf(){
        return this.left==null&&this.right==null;
    }
}
```
###数据的插入
```
//递归实现，判断值的大小，决定放树的左边还是右边
public void insert(T data) {
    if (data==null)
        throw new RuntimeException("data can\'Comparable be null !");
    //插入操作
    root=insert(data,root);
}

/**
 * 插入操作,递归实现
 * @param data
 * @param p
 * @return
 */
private BinaryNode<T> insert(T data,BinaryNode<T> p){
    if(p==null){
        p=new BinaryNode<>(data,null,null);
        return p;
    }
    //比较插入结点的值，决定向左子树还是右子树搜索
    int compareResult=data.compareTo(p.data);
    if (compareResult<0){//左
        p.left=insert(data,p.left);
    }else if(compareResult>0){//右
        p.right=insert(data,p.right);
    }else {
        ;//已有元素就没必要重复插入了
    }
    return p;
}
```
###节点删除
需要考虑三种情况：
1.如果是叶子节点q，可以直接删除
2.如果是删除的节点q有一个孩子节点，则应该调整要被删除的父结点(p.left 或 p.right)指向被删除结点的孩子结点（q.left 或 q.right）
3.如果要删除的结点q拥有两个孩子结点，则删除策略是用q的右子树的最小的数据替代要被删除结点的数据，并递归删除用于替换的结点(此时该结点已为空)，此时二叉查找树的结构并不会被打乱，其特性仍旧生效。采用这样策略的主要原因是右子树的最小结点的数据替换要被删除的结点后可以满足维持二叉查找树的结构和特性，又因为右子树最小结点不可能有左孩子，删除起来也相对简单些。
```
//代码实现
//递归实现
@Override
public void remove(T data) {
  if(data==null)
      throw new RuntimeException("data can\'Comparable be null !");
  //删除结点
  root=remove(data,root);
}

/**
* 分3种情况
* 1.删除叶子结点(也就是没有孩子结点)
* 2.删除拥有一个孩子结点的结点(可能是左孩子也可能是右孩子)
* 3.删除拥有两个孩子结点的结点
* @param data
* @param p
* @return
*/
private BinaryNode<T> remove(T data,BinaryNode<T> p){
  //没有找到要删除的元素,递归结束
  if (p==null){
      return p;
  }
  int compareResult=data.compareTo(p.data);
  if (compareResult<0){//左边查找删除结点
      p.left=remove(data,p.left);
  }else if (compareResult>0) {
      p.right=remove(data,p.right);
  }else if (p.left!=null&&p.right!=null){//已找到结点并判断是否有两个子结点(情况3)
      //中继替换，找到右子树中最小的元素并替换需要删除的元素值
      p.data = findMin( p.right ).data;
      //移除用于替换的结点
      p.right = remove( p.data, p.right );
  }else {
      //拥有一个孩子结点的结点和叶子结点的情况
      p=(p.left!=null)? p.left : p.right;
  }

  return p;//返回该结点
}
//非递归实现
 public T removeUnrecure(T data){
     if (data==null){
         throw new RuntimeException("data can\'Comparable be null !");
     }
     //从根结点开始查找
     BinaryNode<T> current =this.root;
     //记录父结点
     BinaryNode<T> parent=this.root;
     //判断左右孩子的flag
     boolean isLeft=true;


     //找到要删除的结点
     while (data.compareTo(current.data)!=0){
         //更新父结点记录
         parent=current;
         int result=data.compareTo(current.data);

         if(result<0){//从左子树查找
             isLeft=true;
             current=current.left;
         }else if(result>0){//从右子树查找
             isLeft=false;
             current=current.right;
         }
         //如果没有找到,返回null
         if (current==null){
             return null;
         }
     }

     //----------到这里说明已找到要删除的结点

     //删除的是叶子结点
     if (current.left==null&&current.right==null){
         if (current==this.root){
             this.root=null;
         } else if(isLeft){
             parent.left=null;
         }else {
             parent.right=null;
         }
     }
     //删除带有一个孩子结点的结点,当current的right不为null
     else if (current.left==null){
         if (current==this.root){
             this.root=current.right;
         }else if(isLeft){//current为parent的左孩子
             parent.left=current.right;
         }else {//current为parent的右孩子
             parent.right=current.right;
         }
     }
     //删除带有一个孩子结点的结点,当current的left不为null
     else if(current.right==null){
         if (current==this.root){
             this.root=current.left;
         }else if (isLeft){//current为parent的左孩子
             parent.left=current.left;
         }else {//current为parent的右孩子
             parent.right=current.left;
         }
     }
     //删除带有两个孩子结点的结点
     else {
         //找到当前要删除结点current的右子树中的最小值元素
         BinaryNode<T> successor= findSuccessor(current);

         if(current == root) {
             this.root = successor;
         } else if(isLeft) {
             parent.left = successor;
         } else{
             parent.right = successor;
         }
         //把当前要删除的结点的左孩子赋值给successor
         successor.left = current.left;
     }
     return current.data;
 }

 /**
  * 查找中继结点--右子树最小值结点
  * @param delNode 要删除的结点
  * @return
  */
 public BinaryNode<T> findSuccessor(BinaryNode<T> delNode) {
     BinaryNode<T> successor = delNode;
     BinaryNode<T> successorParent = delNode;
     BinaryNode<T> current = delNode.right;

     //不断查找左结点,直到为空,则successor为最小值结点
     while(current != null) {
         successorParent = successor;
         successor = current;
         current = current.left;
     }
     //如果要删除结点的右孩子与successor不相等,则执行如下操作(如果相当,则说明删除结点)
     if(successor != delNode.right) {
         successorParent.left = successor.right;
         //把中继结点的右孩子指向当前要删除结点的右孩子
         successor.right = delNode.right;
     }
     return successor;
 }
```
###树的遍历
####前序遍历
```

```
####中序遍历
```

```
####后序遍历
```

```