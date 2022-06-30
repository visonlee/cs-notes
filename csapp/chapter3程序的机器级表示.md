## gcc编译选项
- -Og 告诉编译器生成符合原始C代码整体结构的机器及代码


***

## Sizes of C data types in x86-64. With a 64-bit machine, pointers are 8 bytes long
| C declaration | Intel data type | Assembly-code suffifix | Size (bytes) |
| :----: | :----: | :----: | :----: |
|char    | | Byte           | b | 1 |
|short   | Word             | w | 2 |
|int     | Double word      | l | 4 |
|long    | Quad word        | q | 8 |
|char *  | Quad word        | q | 8 |
|float   | Single precision | s | 4 |
|double  | Double precision | l | 8 |
