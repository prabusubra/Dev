# Stream Operators:-

  - ## teeing():-

```
  public class CC123 {
    public static void main(String[] args) {

        List<EmployeeA> emp = List.of(new EmployeeA(1, "AAA", "HR", 100),
                new EmployeeA(2, "BBB", "HR", 200),
                new EmployeeA(3, "CCC", "Fin", 300),
                new EmployeeA(4, "DDD", "Fin", 400),
                new EmployeeA(5, "DDD", "Fin", 500),
                new EmployeeA(6, "EEE", "Marketing", 300),
                new EmployeeA(7, "FFF", "Marketing", 400),
                new EmployeeA(8, "GGG", "Marketing", 600));

        //Find max and min salary
        var res1 = emp.stream()
                .collect(Collectors
                        .teeing(Collectors.maxBy(Comparator.comparingInt(EmployeeA::getSalary)),
                                Collectors.minBy(Comparator.comparingInt(EmployeeA::getSalary)),
                                (a, b) -> "Max is "+a.get() +" Min is "+b.get()));

        System.out.println("Result 1 : "+res1);

        //Group by deparment and find max and min salary
        var res2 = emp.stream()
                .collect(Collectors.groupingBy( d -> d.getDept(), Collectors
                        .teeing(Collectors.maxBy(Comparator.comparingInt(EmployeeA::getSalary)),
                                Collectors.minBy(Comparator.comparingInt(EmployeeA::getSalary)),
                                (a, b) -> "Max is "+a.get() +" Min is "+b.get())));

        System.out.println("Result 2 : "+res2);

        //Department is Fin and salary is greater than 300
        List<List<EmployeeA>> emp1 = emp.stream()
                .collect(Collectors
                        .teeing(Collectors.filtering(d -> d.getDept().equals("Fin"), Collectors.toList()),
                                Collectors.filtering(d -> d.getSalary() > 300, Collectors.toList()),
                                List::of));

        System.out.println("Result 3 : "+emp1);

    }
}
```
---
```
@Data
@AllArgsConstructor
class EmployeeA {
    private final long id;
    private final String name;
    private final String dept;
    private final int salary;
}
```
---
