> 多看看底层实现

Test.class

    class Test {

        public synchronized void testMethod(){
            System.out.println("this is method");
        }

        public void testBlock(){
            synchronized (this){
                System.out.println("this is block");
            }
        }

        public static synchronized void testStaticMethod(){
            System.out.println("this is static method");
        }
    }

用javap -v反编译之后得到的字节码

    class Test
      minor version: 0
      major version: 51
      flags: ACC_SUPER
    Constant pool:
       #1 = Methodref          #10.#30        // java/lang/Object."<init>":()V
       #2 = Fieldref           #31.#32        // java/lang/System.out:Ljava/io/PrintStream;
       #3 = String             #33            // this is method
       #4 = Methodref          #34.#35        // java/io/PrintStream.println:(Ljava/lang/String;)V
       #5 = String             #36            // this is block
       #6 = String             #37            // this is static method
       #7 = String             #38            // adasdad
       #8 = Fieldref           #9.#39         // Test.xx:Ljava/lang/String;
       #9 = Class              #40            // Test
      #10 = Class              #41            // java/lang/Object
      #11 = Utf8               xx
      #12 = Utf8               Ljava/lang/String;
      #13 = Utf8               <init>
      #14 = Utf8               ()V
      #15 = Utf8               Code
      #16 = Utf8               LineNumberTable
      #17 = Utf8               LocalVariableTable
      #18 = Utf8               this
      #19 = Utf8               LTest;
      #20 = Utf8               testMethod
      #21 = Utf8               testBlock
      #22 = Utf8               StackMapTable
      #23 = Class              #40            // Test
      #24 = Class              #41            // java/lang/Object
      #25 = Class              #42            // java/lang/Throwable
      #26 = Utf8               testStaticMethod
      #27 = Utf8               <clinit>
      #28 = Utf8               SourceFile
      #29 = Utf8               Test.java
      #30 = NameAndType        #13:#14        // "<init>":()V
      #31 = Class              #43            // java/lang/System
      #32 = NameAndType        #44:#45        // out:Ljava/io/PrintStream;
      #33 = Utf8               this is method
      #34 = Class              #46            // java/io/PrintStream
      #35 = NameAndType        #47:#48        // println:(Ljava/lang/String;)V
      #36 = Utf8               this is block
      #37 = Utf8               this is static method
      #38 = Utf8               adasdad
      #39 = NameAndType        #11:#12        // xx:Ljava/lang/String;
      #40 = Utf8               Test
      #41 = Utf8               java/lang/Object
      #42 = Utf8               java/lang/Throwable
      #43 = Utf8               java/lang/System
      #44 = Utf8               out
      #45 = Utf8               Ljava/io/PrintStream;
      #46 = Utf8               java/io/PrintStream
      #47 = Utf8               println
      #48 = Utf8               (Ljava/lang/String;)V
    {
      Test();
        descriptor: ()V
        flags:
        Code:
          stack=1, locals=1, args_size=1
             0: aload_0
             1: invokespecial #1                  // Method java/lang/Object."<init>":()V
             4: return
          LineNumberTable:
            line 3: 0
          LocalVariableTable:
            Start  Length  Slot  Name   Signature
                0       5     0  this   LTest;

      public synchronized void testMethod();
        descriptor: ()V
        flags: ACC_PUBLIC, ACC_SYNCHRONIZED
        Code:
          stack=2, locals=1, args_size=1
             0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
             3: ldc           #3                  // String this is method
             5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
             8: return
          LineNumberTable:
            line 7: 0
            line 8: 8
          LocalVariableTable:
            Start  Length  Slot  Name   Signature
                0       9     0  this   LTest;

      public void testBlock();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
          stack=2, locals=3, args_size=1
             0: aload_0
             1: dup
             2: astore_1
             3: monitorenter
             4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
             7: ldc           #5                  // String this is block
             9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
            12: aload_1
            13: monitorexit
            14: goto          22
            17: astore_2
            18: aload_1
            19: monitorexit
            20: aload_2
            21: athrow
            22: return
          Exception table:
             from    to  target type
                 4    14    17   any
                17    20    17   any
          LineNumberTable:
            line 11: 0
            line 12: 4
            line 13: 12
            line 14: 22
          LocalVariableTable:
            Start  Length  Slot  Name   Signature
                0      23     0  this   LTest;
          StackMapTable: number_of_entries = 2
            frame_type = 255 /* full_frame */
              offset_delta = 17
              locals = [ class Test, class java/lang/Object ]
              stack = [ class java/lang/Throwable ]
            frame_type = 250 /* chop */
              offset_delta = 4

      public static synchronized void testStaticMethod();
        descriptor: ()V
        flags: ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
        Code:
          stack=2, locals=0, args_size=0
             0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
             3: ldc           #6                  // String this is static method
             5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
             8: return
          LineNumberTable:
            line 17: 0
            line 18: 8

      static {};
        descriptor: ()V
        flags: ACC_STATIC
        Code:
          stack=1, locals=0, args_size=0
             0: ldc           #7                  // String adasdad
             2: putstatic     #8                  // Field xx:Ljava/lang/String;
             5: return
          LineNumberTable:
            line 5: 0
    }


从字节码上可以看出，凡是被Synchronized关键字修饰的代码块都会有monitorenter和monitorexit两个字节码命令,而被synchronized修饰的方法则会有ACC_SYNCHRONIZED标记。

synchronized的锁机制原理简单描述如下：
1. 每个线程走到monitorenter命令的时候会去获取对应的monitor
2. 如果当前monitor没有被任何线程获取获取已经被该线程获取过了则成功获取锁，并将当前monitor计数器加一
3. 执行monitorexit时则将对应的计数器减一


