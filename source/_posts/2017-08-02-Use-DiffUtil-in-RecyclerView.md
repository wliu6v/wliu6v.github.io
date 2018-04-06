---
title: 【译】在 RecyclerView 中使用 DiffUtil
date: 2017-08-02 14:32:12
tags: [Android]
---

[原文：Using DiffUtil in Android RecyclerView](https://medium.com/@iammert/using-diffutil-in-android-recyclerview-bdca8e4fbb00)

> 使用 DiffUtil 从而避免使用 notifyDataSetChanged 更新 RecyclerView 中的数据

<!--more-->

> DiffUtil 是一个用来计算两个列表之间差别的工具类，它会输出一个将 list1 转化为 list2 的操作列表。这通常用来计算 RecyclerView Adapter 的更新逻辑

很多时候我们的数据列表是完全变化的。我们会将新的列表赋值给 RecyclerView Adapter，并调用 notifyDataSetChanged 来更新 Adapter。但是 NotifyDataSetChanged 操作花销高昂， 而使用 DiffUtil 类就可以完美的解决这个问题。

让我们创建一个用例。假设我们有一个人员列表，我们可以随机添加人员，然后按照年龄将其排序，并更新到 RecyclerView 中。

我将用静态数据类使得示例代码尽量简洁：

## DataProvider.java
``` JAVA
public class DataProvider {

    public static List<Person> getOldPersonList(){
        List<Person> persons = new ArrayList<>();
        persons.add(new Person(1, 20, "John"));
        persons.add(new Person(2, 12, "Jack"));
        persons.add(new Person(3, 8, "Michael"));
        persons.add(new Person(4, 19, "Rafael"));
        return persons;
    }

    public static List<Person> sortByAge(List<Person> oldList){
        Collections.sort(oldList, new Comparator<Person>() {
            @Override
            public int compare(Person person, Person person2) {
                return person.age - person2.age;
            }
        });
        return oldList;
    }
}
```

现在我们的主角登场了。DiffUtil.Callback 是一个抽象类，它有 4 个抽象方法和 1 个非抽象方法。接下来让我们简单介绍一下。

## DiffUtil.Callback方法

- `getOldListSize()` : 返回旧列表的大小;
- `getNewListSize()` : 返回新列表的大小;
- `areItemsTheSame(int oldItemPosition, int newItemPosition)` : DiffUtil 使用这个方法来判断两个对象是否表示相同的 Item。如果 Item 有唯一 ID ，那么该方法应使用 ID 判断这些 Item 是否相同。【译者注：此处返回值将控制 Item Added/Moved/Removed 等行为】
- `areContentsTheSame(int oldItemPosition, int newItemPosition)` : 检查两个项目是否具有相同的数据。您可以根据您的 UI 更改其行为。只有当 `areItemsTheSame` 返回 true 时，此方法才被 DiffUtil 调用。【译者注：此处返回 false 将触发 Item Changed 行为】
- `getChangePayload(int oldItemPosition, int newItemPosition)` : 当 areItemTheSame 返回 true 并且 isContentsTheSame 返回 false 时，DiffUtil 调用此方法以获取有关该更改的有效负载。在当前的实例中并没有使用 payload object。但是你可以通过[这个示例](https://proandroiddev.com/diffutil-is-a-must-797502bc1149)进行尝试。
	
## MyDiffUtilCallback.java
``` JAVA
public class MyDiffCallback extends DiffUtil.Callback{

    List<Person> oldPersons;
    List<Person> newPersons;

    public MyDiffCallback(List<Person> newPersons, List<Person> oldPersons) {
        this.newPersons = newPersons;
        this.oldPersons = oldPersons;
    }

    @Override
    public int getOldListSize() {
        return oldPersons.size();
    }

    @Override
    public int getNewListSize() {
        return newPersons.size();
    }

    @Override
    public boolean areItemsTheSame(int oldItemPosition, int newItemPosition) {
        return oldPersons.get(oldItemPosition).id == newPersons.get(newItemPosition).id;
    }

    @Override
    public boolean areContentsTheSame(int oldItemPosition, int newItemPosition) {
        return oldPersons.get(oldItemPosition).equals(newPersons.get(newItemPosition));
    }

    @Nullable
    @Override
    public Object getChangePayload(int oldItemPosition, int newItemPosition) {
        //you can return particular field for changed item.
        return super.getChangePayload(oldItemPosition, newItemPosition);
    }
}
```

RecyclerViewAdapter 的更新方法
``` JAVA
public void updateList(ArrayList<Person> newList) {
    DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new MyDiffCallback(this.persons, newList));
    diffResult.dispatchUpdatesTo(this);
}
```

以上。我们调用 dispactUpdatesTo(RecyclerView.Adapter) 方法，在 Adapter 中执行对应的更改。

## 参考
[https://developer.android.com/reference/android/support/v7/util/DiffUtil.html](https://developer.android.com/reference/android/support/v7/util/DiffUtil.html)
[https://medium.com/@nullthemall/diffutil-is-a-must-797502bc1149#.x7kc55a69](https://medium.com/@nullthemall/diffutil-is-a-must-797502bc1149#.x7kc55a69)