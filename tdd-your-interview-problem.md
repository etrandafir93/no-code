# Make your live interview problem meomrable

In some technical interviews you'll be assigned a small problem which you will need to code live while sharing the screeen. It can be done using your own IDE, but usually they will want you to use a special website for interviews which compiles the code and displays the output.

If you want to standout at this coding problem, you can TDD it. By using TDD (Test Driven Development) you will write a test **before** writing the actual implementation for that piece of logic.

To demonstrate this, we will assume the interviewer gave us a simple FizzBuzz problem and he asked us to solve it on [this](https://www.interviewbit.com/online-java-compiler/)  online java compiler website.
The requirements are simple: You have to iterate through a list of numbers from 1 to 100 and, for each of them, print "Fizz" if the number is a multiple of 3, "Buzz" if the number it s a multiple of 5, "FizzBuzz" if the number is a multiple of both and the number itself if none of the previous conditions were met.

The first instinct would be to start coding everything inside the for loop itself.
```
public static void main(String args[]) {  
  for(int i=1; i<=100; i++) {
    if(i % 3 == 0 && i % 5 == 0) {
      System.out.println("FizzBuzz");
    } 
    //.. and so on  
  }
}
```
But this piece of code will be very hard to test because the output is printed in the console. We can change our approach and extract the logic into a separate method, such as in the example below:
```
public static void main(String args[]) {
  for(int i=1; i<=100; i++) {
    System.out.println(getOutput(i)) 
  }
}

public static String getOutput(int i) {
  if(i % 3 == 0 && i % 5 == 0) {
    return "FizzBuzz";
  } 
  //.. and so on
}
```
This way we can call the getOutput() method with some specific parameters and check its output. This means we can already write out first 'tests'. 
Of Course, since we are using this online editor, we are not going to write actual junit tests - but what we can to do it's something like this:
```	
public static void main(String args[]) {
  assert getOutput(1).equals("1");
  assert getOutput(3).equals("Fizz");
  assert getOutput(5).equals("Buzz");  
  assert getOutput(15).equals("FizzBuzz");

  for(int i=1; i<=100; i++) {
    System.out.println(getOutput(i));
  }
  // ...
```
PS: if the assert keyword is not working the the online editor you can just replace it with something simple such as *System.out.println(getOutput(3) + " should be 'Fizz'")*

Now you can finish the implementation.

After this, the interviewer will probably add new requirements such as this one: *"the program should print 'Woof' for dividers of 7". What will you do next?* 

(Answer): write a test! 
```
assert getOutput(7).equals("Woof");
```
Sometimes the interviewers are leaving some small 'holes' in the requirements to see if you manage to find and address them. Pa attention to this, try find the areas where the the task is unclear and ask the appropriate questions. In this case you should ask what the output should be for input values such as 21 - should it be "Fizz"? "Woof"? "FizzWoof"?
Once you've got your answers, you can continue with... some new tests! 

```
assert getOutput(3*7).equals("FizzWoof");
assert getOutput(5*7).equals("BuzzWoof");
assert getOutput(3*5*7).equals("FizzBuzzWoof");
```

At this point you're all set and you can continue coding.

## Why would you do this?
- it shows that your code is decoupled and testable
- it will be easier to check the output for certain values
- if you break something when implementing new requirements, you will detect it faster
- you'll notice the holes in the requirement easier

Of course, some T.D.D. evangelists may argue this is not actual test driven development, but I would say it's good enough and pretty clever - for the given context.

## Bonus - my implementation

Since I wrote this article and gave you this FizzBuzz example, I have decided to give it a try myself and wirte my own implementation of it.
I have tried to write the code less imperative (so not many ifs and elses). This was the result:

```
public static void main(String args[]) {
  assert getOutput(4).equals("4");
  assert getOutput(3).equals("Fizz");
  assert getOutput(5).equals("Buzz");
  assert getOutput(3*5).equals("FizzBuzz");
  assert getOutput(3*7).equals("FizzWoof");
  assert getOutput(5*7).equals("BuzzWoof");
  assert getOutput(3*5*7).equals("FizzBuzzWoof");

  for(int i=1; i<=100; i++) {
    System.out.println(getOutput(i));
  }
}

public static String getOutput(int i) {
  String output = "";
  for(Dividers divider : Dividers.values()) {
    if(divider.divides(i)) {
      output += divider.output;
    }
  }
  return output.isEmpty()? i+"" : result;
}

enum Dividers {
  FIZZ("Fizz", 3),
  BUZZ("Buzz", 5),
  WOOF("Woof", 7);

  Dividers(String output, int dividerValue) {
    this.dividerValue = dividerValue;
    this.output = output;
  }
  private final int dividerValue;
  public final String output;

  public boolean divides(int value) {
    return value % dividerValue == 0;
  }
}	
```

As you may have noticed, I'm making use this enum of Dividers where each of them is able to check for itself if the given number is a multipier of it.
In O.O.P. (Object Oriented Programming) terms, this is called **Polymorphism** - but maybe this is a topic for another article.


## Challenge - solve Roman to Integer

As homework, try solving the next problem in a similar - decoupled and testable - way. Code it on [interviewbit](https://www.interviewbit.com/online-java-compiler/) or some other online compiler.

Task: Iterate from 1 to 100, and, for each number, print it's Roman numeral equivalent. 
