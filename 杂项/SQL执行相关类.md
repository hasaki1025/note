# MapperProxy

## SqlSession

## Class mapperInterface

## Map<Method, MapperMethodInvoker> methodCache

### MapperMethodInvoker(接口)

- 实现类

	- DefaultMethodInvoker
	- PlainMethodInvoker

		- MapperMethod

			- SqlCommand

				- name（SQL语句的ID）
				- SqlCommandType(枚举类，用于表示SQL的类型)

					- UNKNOWN, INSERT, UPDATE, DELETE, SELECT, FLUSH

			- MethodSignature

				- 子主题 1

		- invoke方法

			-  return mapperMethod.execute(sqlSession, args);

