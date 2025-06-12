---
tags:
  - 知识笔记/Godot
---
>首先这边用到CharacterBody2D的velocity和global_position属性来测试

#### 测试场景

- Vector2(5,5)设置给global_position,Sprite2D确实和相信的一样，出现在了第二象限5，5的位置
- 固定一个向量Vector2(300,300),然后velocity设置为global_position - 固定的向量，发现负值也确实导致Sprite2D直接往第四象限移动 **所以当你想往某个方向移动一个物体，那么应该使用 需要移动到的点的向量 - 当前的位置的向量**

#### 结论

Godot里的Vector2确实就是数学里的向量，只是API需要注意，一下是本次测试碰到的API需要记录的点

1. 获取两个向量直接的夹角，需要使用angle_to `Vector2(1,0).angle_to(Vector2(0,1))`
2. 所有向量相关的API一定要注意返回值是弧度值还是角度值，以及基于谁的，否则很容易晕，比如`angle_to`返回的是弧度值，但是它返回的是两个向量之间的夹角，而`angle_to_point`返回的也是弧度值，但是它返回的是(b-a).angle()也就是给定的两个向量（后面的-前面的），然后结果向量和X轴的夹角。。。
3. `rad_to_deg`和`deg_to_rag`，前者是**将弧度转为角度**，后者是**将角度转为弧度**，比如 `Vector2(1,0).angle_to(Vector2(0,1))`的夹角明显是90度，但是测试中拿到的值≈1.57，也就是说我需要`rad_to_deg(Vector2(1,0).angle_to(Vector2(0,1)))`才能拿到≈90度
4. `Vector2(1,1).distance_to(Vector2.ZERO)`和`Vector2(1,1).length()`一样;`Vector2(1,1).distance_squared_to(Vector2.ZERO)`和`Vector2(1,1).length_squared()`也一样，所以**length**,**length_squared**默认就是针对Vector2.ZERO的