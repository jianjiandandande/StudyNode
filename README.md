# StudyNode
小型知识点总汇

1.[JAVA中的九种数据类型]
### java中包含了九种数据类型主要包括：boolean,byte,char,short,int,long, float,double,void.
*	boolean 的包裹类型为：Boolean，它的取值只有两种：true,false.
* void 包裹类型为 Void
*	byte 占一个字节，位数为8bit  包裹类型为：Byte，它的取值范围为-2^7～2^7 - 1.
*	char 占两个字节，位数为16bit 包裹类型为：Character，它的取值范围为：0～2^16 - 1
* short 占两个字节，位数为16bit 包裹类型为：Short,它的取值范围为：-2^15～2^15 - 1
* int 占四个字节，位数为32bit 包裹类型为：Integer,它的取值范围为：-2^31～2^31 - 1
* long 占八个字节，位数为64bit 包裹类型为 Long，它的取值范围为：-2^63～2^63- 1
* float 占四个字节，位数为32bit 包裹类型为 Float，它的取值范围为：1.4E-45～3.4028235E38
* double 占八个字节，位数为64bit 包裹类型为 Double，它的取值范围为：4.9E-324～1.7976931348623157E308


2.[java中的switch-case语句]
### switch(A),括号中A的取值只能是整型或者可以转换为整型的数值类型，比如byte、short、int、char、还有[枚举](http://baike.baidu.com/link?url=9VzlTPyBth5SrT03Fk8-eRPMqSxfeq33GYVM0EtHdJ-jSHakcvjhT6MX4yhgDKLxOCFXuqAn4fIGY6t-TGArjpGCm9nNdgt9moM6unpG4BYlzd3AUd3Da3VMViNXemhq)；需要强调的是：
* long类型是不能作用在switch语句上的。
* String类型在java7之前是不能作用在switch语句上的，但从java7开始，String类型可以作用在switch语句上了（这个新特性是在编译器这个层次上实现的）。
### case B：C；case是常量表达式，也就是说B的取值只能是常量（需要定义一个final型的常量）或者int、byte、short、char（比如1、2、3、200000000000（注意了这是整型）），case后的语句可以不用大括号，就是C不需要用大括号包裹着；
