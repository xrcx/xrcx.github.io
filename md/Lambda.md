- @FunctionalInterface 是 Java 8 新加入的一种接口，用于指明该接口类型声明是根据 Java 语言规范定义的函数式接口



### forEach ###
	//New way:
	List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7);
	list.forEach(n -> System.out.println(n));

	//or we can use :: double colon operator in Java 8
	list.forEach(System.out::println);

### map ###
	//Old way:
	List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
	for(Integer n : list) {
	    int x = n * n;
	    System.out.println(x);
	}
	 
	//New way:
	List<Integer> list = Arrays.asList(1,2,3,4,5,6,7);
	list.stream().map((x) -> x*x).forEach(System.out::println);

