##1.list和set

由于数据中存在重复元素，所以使用contains()方法，但是，ArrayList的contains()方法会调用其indexOf()方法，在indexOf()方法里边，有一个for循环，所以，ArrayList的contains()方法的时间复杂度是O(n)

对于HashSet，它的add()方法会自动去重，它调用的是一个map的put方法，其时间复杂度是O(1)
