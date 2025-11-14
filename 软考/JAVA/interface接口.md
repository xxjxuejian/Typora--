```java
Interface Patient{
    （1）
}

class ConcretePatient implements Patient{
    private String name;
    public ConcretePatient(String name){
        this.name = name;
    }
    public String getName(){
        return name;
    }
}
```



抽象类里面的 抽象方法是没有方法体 的，普通的方法是有方法体的

接口里面的方法都是没有方法体的，另外还可以理解为接口中的方法 都是抽象方法，写不写abstract都可以



接口中可以定义方法，那么为什么还要使用抽象类定义方法呢

另外能用抽象类来定义，是不是可以直接用一个普通的类，然后继承呢



**在抽象类中声明一个没有方法体的方法时，必须使用 `abstract`**。

```java

abstract class Piece { // 棋子定义
    protected PieceColor m_color; // 颜色
    protected PiecePos m_pos; // 位置
    public Piece(PieceColor color, PiecePos pos) {
        m_color = color;
        m_pos = pos;
    }
    (1) ; //  public abstract void draw(); 
}


class BlackPiece extends Piece {
    public BlackPiece(PieceColor color, PiecePos pos) {
    	super(color, pos);
    }
    public void draw() { System.out.println("draw a blackpiece"); }
}
```

**一旦类中出现了抽象方法，这个类也必须被声明为 `abstract`。**
