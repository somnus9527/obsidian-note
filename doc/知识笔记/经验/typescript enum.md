---
tags:
  - 知识笔记/经验
---
>[!Tip] typescript enum 可以添加静态方法
> ```typescript
> enum Weekday {
>   Monday,
>   Tuesday,
>   Wednesday,
>   Thursday,
>   Friday,
>   Saturday,
>   Sunday
> }
> 
> namespace Weekday {
>   export function isBusinessDay(day: Weekday) {
>     switch (day) {
>       case Weekday.Saturday:
>       case Weekday.Sunday:
>         return false;
>       default:
>         return true;
>     }
>   }
> }
> ```

