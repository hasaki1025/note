# System类

- ##流

  | `static PrintStream` | `err`        “标准”错误输出流。 |
  | -------------------- | ------------------------------- |
  | `static InputStream` | `in       “标准”输入流。        |
  | `static PrintStream` | `out`        “标准”输出流。     |

- ## 常用方法

  - 重定向流

  | `static void` | `setErr(PrintStream err)`        重新分配“标准”错误输出流。 |
  | ------------- | ----------------------------------------------------------- |
  | `static void` | `setIn(InputStream in)`        重新分配“标准”输入流。       |
  | `static void` | `setOut(PrintStream out)`        重新分配“标准”输出流。     |

  



