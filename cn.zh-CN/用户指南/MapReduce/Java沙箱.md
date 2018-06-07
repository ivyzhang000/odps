# Java沙箱 {#concept_aln_lrf_vdb .concept}

MaxCompute MapReduce 及 UDF 程序在分布式环境中运行时，受到 Java 沙箱的限制（MapReduce 作业的主程序则不受此限制），具体限制如下：

-   不允许直接访问本地文件，只能通过 MaxCompute MapReduce/Graph 提供的接口间接访问。
    -   读取 -resources 选项指定的资源，包括文件、Jar 包和资源表等。
    -   通过 System.out 和 System.err 输出日志信息，可以通过 MaxCompute 客户端的 Log 命令查看日志信息。
-   不允许直接访问分布式文件系统，只能通过 MaxCompute MapReduce/Graph 访问到表的记录。
-   不允许 JNI 调用限制。
-   不允许创建 Java 线程，不允许启动子进程执行 Linux 命令。
-   不允许访问网络，包括获取本地 IP 地址等，都会被禁止。
-   Java 反射限制：suppressAccessChecks 权限被禁止，无法 setAccessible 某个 private 的属性或方法，以达到读取 private 属性或调用 private 方法的目的。

具体的说，在代码中，直接使用下面这些方法会报 access denied 异常。

访问本地文件的相关方法，如下所示：

java.io.File:

```
      public boolean delete()
      public void deleteOnExit()
      public boolean exists()
      public boolean canRead()
      public boolean isFile()
      public boolean isDirectory()
      public boolean isHidden()
      public long lastModified()
      public long length()
      public String[] list()
      public String[] list(FilenameFilter filter)
      public File[] listFiles()
      public File[] listFiles(FilenameFilter filter)
      public File[] listFiles(FileFilter filter)
      public boolean canWrite()
      public boolean createNewFile()
      public static File createTempFile(String prefix, String suffix)
      public static File createTempFile(String prefix,  String suffix,File directory)
      public boolean mkdir()
      public boolean mkdirs()
      public boolean renameTo(File dest)
      public boolean setLastModified(long time)
      public boolean setReadOnly()
```

java.io.RandomAccessFile:

```
      RandomAccessFile(String name, String mode)
      RandomAccessFile(File file, String mode)
```

java.io.FileInputStream:

```
      FileInputStream(FileDescriptor fdObj)
      FileInputStream(String name)
      FileInputStream(File file)
```

java.io.FileOutputStream:

```
      FileOutputStream(FileDescriptor fdObj)
      FileOutputStream(File file)
      FileOutputStream(String name)
      FileOutputStream(String name, boolean append)
```

java.lang.Class:

```
public ProtectionDomain getProtectionDomain()
```

java.lang.ClassLoader:

```
    ClassLoader()
    ClassLoader(ClassLoader parent)
```

java.lang.Runtime:

```
    public Process exec(String command)
    public Process exec(String command, String envp[])
    public Process exec(String cmdarray[])
    public Process exec(String cmdarray[], String envp[])
    public void exit(int status)
    public static void runFinalizersOnExit(boolean value)
    public void addShutdownHook(Thread hook)
    public boolean removeShutdownHook(Thread hook)
    public void load(String lib)
    public void loadLibrary(String lib)
```

java.lang.System:

```
    public static void exit(int status)
    public static void runFinalizersOnExit(boolean value)
    public static void load(String filename)
    public static void loadLibrary( String libname)
    public static Properties getProperties()
    public static void setProperties(Properties props)
    public static String getProperty(String key) // 只允许部分key可以访问
    public static String getProperty(String key, String def) // 只允许部分key可以访问
    public static String setProperty(String key, String value)
    public static void setIn(InputStream in)
    public static void setOut(PrintStream out)
    public static void setErr(PrintStream err)
    public static synchronized void setSecurityManager(SecurityManager s)
```

System.getProperty 允许的 key 列表，如下所示：

```
    java.version
    java.vendor
    java.vendor.url
    java.class.version
    os.name
    os.version
    os.arch
    file.separator
    path.separator
    line.separator
    java.specification.version
    java.specification.vendor
    java.specification.name
    java.vm.specification.version
    java.vm.specification.vendor
    java.vm.specification.name
    java.vm.version
    java.vm.vendor
    java.vm.name
    file.encoding
    user.timezone
```

java.lang.Thread:

```
    Thread()
    Thread(Runnable target)
    Thread(String name)
    Thread(Runnable target, String name)
    Thread(ThreadGroup group, ...)
    public final void checkAccess()
    public void interrupt()
    public final void suspend()
    public final void resume()
    public final void setPriority (int newPriority)
    public final void setName(String name)
    public final void setDaemon(boolean on)
    public final void stop()
    public final synchronized void stop(Throwable obj)
    public static int enumerate(Thread tarray[])
    public void setContextClassLoader(ClassLoader cl)
```

java.lang.ThreadGroup:

```
    ThreadGroup(String name)
    ThreadGroup(ThreadGroup parent, String name)
    public final void checkAccess()
    public int enumerate(Thread list[])
    public int enumerate(Thread list[], boolean recurse)
    public int enumerate(ThreadGroup list[])
    public int enumerate(ThreadGroup list[], boolean recurse)
    public final ThreadGroup getParent()
    public final void setDaemon(boolean daemon)
    public final void setMaxPriority(int pri)
    public final void suspend()
    public final void resume()
    public final void destroy()
    public final void interrupt()
    public final void stop()
```

java.lang.reflect.AccessibleObject:

```
    public static void setAccessible(...)
    public void setAccessible(...)
```

java.net.InetAddress:

```
    public String getHostName()
    public static InetAddress[] getAllByName(String host)
    public static InetAddress getLocalHost()
```

java.net.DatagramSocket:

```
public InetAddress getLocalAddress()
```

java.net.Socket:

```
Socket(...)
```

java.net.ServerSocket:

```
    ServerSocket(...)
    public Socket accept()
    protected final void implAccept(Socket s)
    public static synchronized void setSocketFactory(...)
    public static synchronized void setSocketImplFactory(...)
```

java.net.DatagramSocket:

```
    DatagramSocket(...)
    public synchronized void receive(DatagramPacket p)
```

java.net.MulticastSocket:

```
MulticastSocket(...)
```

java.net.URL:

```
    URL(...)
    public static synchronized void setURLStreamHandlerFactory(...)
    java.net.URLConnection
    public static synchronized void setContentHandlerFactory(...)
    public static void setFileNameMap(FileNameMap map)
```

java.net.HttpURLConnection:

```
    public static void setFollowRedirects(boolean set)
    java.net.URLClassLoader
    URLClassLoader(...)
```

java.security.AccessControlContext:

```
    public AccessControlContext(AccessControlContext acc, DomainCombiner combiner)
    public DomainCombiner getDomainCombiner()
```

