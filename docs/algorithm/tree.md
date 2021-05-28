# tree

堆其实就是一种完全二叉树，最常用的存储方式就是数组。


## 二叉树的遍历

### 1、先序遍历
```php

<?php
/**
 * Definition for a binary tree node.
 * class TreeNode {
 *     public $val = null;
 *     public $left = null;
 *     public $right = null;
 *     function __construct($val = 0, $left = null, $right = null) {
 *         $this->val = $val;
 *         $this->left = $left;
 *         $this->right = $right;
 *     }
 * }
 */
class Solution {
    /**
     *
     * @param $root Node
     * @return array
     */
      // 非递归
        // 前序遍历 根节点→左子树→右子树
    function preorderTraversal($root){
        $arr=[];
        $stack= new SplStack();
         while(!$stack->isEmpty()||$root!=null){
             while ($root!=null){
                 $stack->push($root);
                 $arr[]=$root->val;
                 $root=$root->left;
             }
             while ($root==null&&!$stack->isEmpty()){
                 $root=$stack->pop()->right;
             }
         }

         return $arr;
    }
    // 递归
    // 前序遍历
    public function pre_order($root) {
        if ($root != null) {
            echo $root->value . " ";        // 根
            if ($root->left != null) {
                $this->pre_order($root->left);     //递归遍历左树
            }
            if ($root->right != null) {
                $this->pre_order($root->right);    //递归遍历右树
            }
        }
    }
    
}

class Node {
    public $val;
    public $left;
    public $right;
}

$a = new Node();
$b = new Node();
$c = new Node();
$d = new Node();
$e = new Node();
$f = new Node();

$a->val = "A";
$b->val = "B";
$c->val = "C";
$d->val = "D";
$e->val = "E";
$f->val = "F";

$a->left = $b;
$a->right = $c;
$b->left = $d;
$b->right = $e;
$c->left = $f;

$bst = new Solution();
var_dump($bst->preorderTraversal($a));


```

### 2、中序遍历
```php

<?php
/**
 * Definition for a binary tree node.
 * class TreeNode {
 *     public $val = null;
 *     public $left = null;
 *     public $right = null;
 *     function __construct($val = 0, $left = null, $right = null) {
 *         $this->val = $val;
 *         $this->left = $left;
 *         $this->right = $right;
 *     }
 * }
 */
class Solution {
 
     // 非递归
    function postorderTraversal($root){
        $arr=[];
        $stack= new SplStack();
        while(!$stack->isEmpty()||$root!=null){
            while ($root!=null){
                $stack->push($root);
                $root=$root->left;
            }
            while ($root==null&&!$stack->isEmpty()){
                $root=$stack->pop();
                $arr[]=$root->val;
                 $root=$root->right;
             }

        }

        return $arr;
    }
    // 递归
    // 中序遍历
    public function in_order($root) {
        if ($root != null) {
            if ($root->left != null) {
                $this->in_order($root->left);  // 递归遍历左树
            }
            echo $root->value . " ";
            if ($root->right != null) {
                $this->in_order($root->right); // 递归遍历右树
            }
        }
    }
}

class Node {
    public $val;
    public $left;
    public $right;
}

$a = new Node();
$b = new Node();
$c = new Node();
$d = new Node();
$e = new Node();
$f = new Node();

$a->val = "A";
$b->val = "B";
$c->val = "C";
$d->val = "D";
$e->val = "E";
$f->val = "F";

$a->left = $b;
$a->right = $c;
$b->left = $d;
$b->right = $e;
$c->left = $f;

$bst = new Solution();
var_dump($bst->inorderTraversal($a));



```
###  3、后序遍历


```php

<?php
/**
 * Definition for a binary tree node.
 * class TreeNode {
 *     public $val = null;
 *     public $left = null;
 *     public $right = null;
 *     function __construct($val = 0, $left = null, $right = null) {
 *         $this->val = $val;
 *         $this->left = $left;
 *         $this->right = $right;
 *     }
 * }
 */
class Solution {

    function postorderTraversal($root){
        $arr=[];
        $stack= new SplStack();
        $pre = null;    //记录之前访问过的结点
        while(!$stack->isEmpty()||$root!=null){
            while ($root!=null){
                $stack->push($root);
                $root=$root->left;
            }
            $root = $stack->top();
            if($root->right==null||$root->right==$pre){
                $pre=$root;
                $arr[]=$stack->pop()->val;
                $root=null;
            }else{
                $root=$root->right;
            }

        }

        return $arr;
    }
         // 递归
        // 后序遍历
        public function post_order($root) {
            if ($root != null) {
                if ($root->left != null) {
                    $this->post_order($root->left);  // 递归遍历左树
                }
                if ($root->right != null) {
                    $this->post_order($root->right); // 递归遍历右树
                }
                echo $root->value . " ";    // 根
            }
        }
}

class Node {
    public $val;
    public $left;
    public $right;
}

$a = new Node();
$b = new Node();
$c = new Node();
$d = new Node();
$e = new Node();
$f = new Node();

$a->val = "A";
$b->val = "B";
$c->val = "C";
$d->val = "D";
$e->val = "E";
$f->val = "F";

$a->left = $b;
$a->right = $c;
$b->left = $d;
$b->right = $e;
$c->left = $f;

$bst = new Solution();
var_dump($bst->postorderTraversal($a));


```
### 4 层次遍历
```php
<?php
class Node {
    public $value;
    public $left;
    public $right;
}
class BT {
    // 非递归
    public function levelOrder($root) {
        if ($root == null) {
            return;
        }
        $node = $root;
        $queue = array();
        array_push($queue, $node);  // 根节点入队
        while (!empty($queue)) {    // 持续输出节点，直到队列为空
            $node = array_shift($queue);    // 队首元素出队
            echo $node->value . " ";
            // 左节点先入队
            if ($node->left != null) {
                array_push($queue, $node->left);
            }
            // 然后右节点入队
            if ($node->right != null) {
                array_push($queue, $node->right);
            }
        }
    }

    // 递归
    // 获取树的层数（最大深度）
    function getDepth($root) {
        if ($root == null) { // 节点为空
            return 0;
        }
        if ($root->left == null && $root->right == null) { // 只有根节点
            return 1;
        }

        $left_depth = $this->getDepth($root->left);
        $right_depth = $this->getDepth($root->right);

        return ($left_depth > $right_depth ? $left_depth : $right_depth) + 1;
//        return $left_depth > $right_depth ? ($left_depth + 1) : ($right_depth + 1);
    }

    public function level_order($root) {
        // 空树或层级不合理
        $depth = $this->getDepth($root);
        if ($root == null || $depth < 1) {
            return;
        }
        for ($i = 1; $i <= $depth; $i++) {
            $this->printTree($root, $i);
        }
    }

    public function printTree($root, $level) {
        // 空树或层级不合理
        if ($root == null || $level < 1) {
            return;
        }
        if ($level == 1) {
            echo $root->value;
        }
        $this->printTree($root->left, $level - 1);
        $this->printTree($root->right, $level - 1);
    }
}

// 测试
$a = new Node();
$b = new Node();
$c = new Node();
$d = new Node();
$e = new Node();
$f = new Node();

$a->value = "A";
$b->value = "B";
$c->value = "C";
$d->value = "D";
$e->value = "E";
$f->value = "F";

$a->left = $b;
$a->right = $c;
$b->left = $d;
$b->right = $e;
$c->left = $f;

$bst = new BT();
echo "----广度优先----";
echo "</br>";
echo "非递归：";
$bst->levelOrder($a);
echo "</br>";
echo "递归：";
$bst->level_order($a);

PHP用递归、非递归方式实现二叉树的层次遍历
```