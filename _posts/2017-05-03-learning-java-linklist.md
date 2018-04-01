#**用Java实现单向链表**

***
##**说明**

>  1. 实现的链表为最简单的单向链表，只作为复习和理解链表的知识，实际开发中Java有现成的集合类使用
>  2. 链表存储的数据为`String`类型，同时也可以修改为其它类型或自定义类型，但是自定义类要实现 `compare()`方法，用来链表实现类里面的比较用。
>  3. 代码参考自李兴华老师的Java视频，结构使用内部类的方式来实现，
>  4. 链表实现的方法主要有：增、删、查、获取链表元素个数、将链表转为数组、对链表的一些判断

##**代码**

Link（链表)类

```java
/**
 * 链表的实现
 * Created by Veir on 2017/3/16.
 */
class Link{
    private class Node{
        //链表节点
        private String data;
        private Node next;

        public Node() {
        }

        public Node(String data) {
            this.data = data;
        }

        public String getData() {
            return data;
        }

        public void setData(String data) {
            this.data = data;
        }

        public Node getNext() {
            return next;
        }

        public void setNext(Node next) {
            this.next = next;
        }

        public void addNode(Node newNode){
            if (this.getNext() == null){
                this.setNext(newNode);
            }else {
                this.getNext().addNode(newNode);
            }
        }
        public boolean containsNode(String data){
            if (data.equals(this.getData())){
                return true;
            }else{
                if (this.getNext() != null){
                    return this.getNext().containsNode(data);
                }else {
                    return false;
                }
            }
        }

        public String getNode(int index) {
            if (Link.this.foot ++ == index) {
                return this.getData();
            }else {
                return this.getNext().getNode(index);
            }

        }

        public void setNode(int index, String data) {
            if (Link.this.foot ++ == index) {
                this.setData(data);
            } else {
                this.getNext().setNode(index, data);
            }
        }

        public void removeNode(int index) {
            if (Link.this.foot ++ == index - 1) {
                this.setNext(this.getNext().getNext());
            }else {
                this.getNext().removeNode(index);
            }

        }

        public void toArrayNode() {
            Link.this.strArray[Link.this.foot++] = this.getData();
            if (this.getNext() != null){
                this.getNext().toArrayNode();
            }
        }
    }
    /****************************以上为内部类*******************************************/
    private Node root;//根结点
    private int count = 0;//数据个数
    private int foot = 0;//查询索引
    private String[] strArray;//返回的数组

    /**
     * 链表——添加新数据
     * @param data 保存的对象
     */
    public  void add(String data){
        if (data == null){
            return;
        }
        Node newNode = new Node(data);
        if (this.root == null){//当前没有根节点存在，需要任命第一个结点为根结点
            this.root = newNode;
        }else {//已存在根结点，则交由Node类存储下一个结点
            this.root.addNode(newNode);
        }
        this.count++;
    }

    /**
     * 获取当前链表的数据大小
     * @return 数据个数
     */
    public  int size(){
        return count;
    }

    /**
     * 判断当前链表是否为空
     * @return
     */
    public boolean isEmpty(){
        return this.count == 0;
    }

    /**
     * 判断链表中是否有某个元素
     * @param data
     * @return
     */
    public boolean contains(String data){
        if (data == null || this.root == null){
            return false;
        }
        return this.root.containsNode(data);
    }
    /**
     * 获取索引值对于的元素
     * @param index
     * @return
     */
    public String get(int index){
        if (index > this.count){
            return null;
        }
        this.foot = 0;
        return this.root.getNode(index);
    }

    /**
     * 修改某个索引值的元素
     * @param index 索引值
     * @param data 要修改的元素
     */
    public void set(int index, String data){
        if (index > this.count || data == null){
            return;
        }
        this.foot = 0;
        this.root.setNode(index, data);
    }

    /**
     * 删除某个索引对应的元素
     * @param index
     */
    public void remove(int index) {
        if (this.isEmpty() || index > this.count) {
            return;
        }
        if (index == 0){
            this.root = this.root.getNext();
        }else {
            this.foot = 0;
            this.root.removeNode(index);
        }
        this.count --;
    }

    /**
     * 将链表转换为数组，以作遍历用
     * @return
     */
    public String[] toArray(){
        if (this.isEmpty()){
            return null;
        }
        this.foot = 0;
        strArray = new String[this.count];
        this.root.toArrayNode();
        return strArray;
    }
}
```
##**测试代码**

```java
/**
 * 测试代码
 * Created by Veir on 2017/3/16.
 */
public class LinkDemo {
    public static void main(String[] args) {
        Link link= new Link();
        link.add("头结点");
        link.add("第一个结点");
        link.add("第二个结点");
        link.add(null);
        link.add("第三个结点");
//        System.out.println(link.get(4));
//        link.set(2, "数据被修改");
//        link.remove(2);
//        System.out.println(link.get(4));
//        System.out.println(link.size());
        String[] array = link.toArray();
        for (String str : array) {
            System.out.println(str);
        }
    }

   /* public static void print(Node root){
        //递归打印
        if (root == null){
            return ;
        }
        System.out.println(root.getData());
        print(root.getNext());
    }*/


}
```

