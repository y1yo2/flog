# java的字符和字节



大家都清楚一个字节是8位，一个字符有多少字节由编码格式决定。
那java的char和byte是什么关系？
String里面的char，单个char，都是默认用unicode编码的，占用两个字节。
char a = '\uffff'; //Highest value that char can take - 65535
byte b = (byte)a; //Casting a 16-bit value into 8-bit data type...! Isn't data lost here?
// b = 0xFF
char c = (char)b; //Let's get the value back
//Converting a byte to a char is considered a special conversion. It actually performs TWO conversions. First, the byte is SIGN-extended (the new high order bits are copied from the old sign bit) to an int (a normal widening conversion). Second, the int is converted to a char with a narrowing conversion.
// c = 0xFFFF
int d = (int)c;
// d = 0x0000FFFF
System.out.println(d); //65535... how?

但是将String转成byte[]，编码格式是可以自定义的。
如果使用默认方法 getBytes() 将String转成byte[]，
首先会查找file.encoding，如果没有，则使用UTF-8，如果报错，则使用 ISO-8859-1 
