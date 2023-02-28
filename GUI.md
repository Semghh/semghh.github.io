# 1、JFrame类

```java
JFrame jframe = new JFrame("Title name");
jframe.setSize(250,150);
jframe.setIconImage(Image image);  //传入一个Image类对象
jframe.setVisible(Boolean bool)；  //设置可见性
jframe.add(Component component);   //添加组件
jframe.setLayout(LayoutManager manager)； //传入 布局
jframe.setLocation(int x,int y)；  // swing窗口在屏幕中默认显示位置
jframe.setLocationRelativeTo(null) //默认居中
```

# 2、ImageIcon类

```java
ImageIcon icon = new ImageIcon(String filename,String description);//

@Return Image
icon.getImage();           //返回 Image对象
```

