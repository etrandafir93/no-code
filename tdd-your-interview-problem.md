# Make your live interview problem meomrable

In some technical interviews you'll be assigned a small problem which you will need to code live, by sharing the screeen and using your own IDE or using some special website for interviews.

If you want to standout at this coding problem, you can TDD it. By using TDD (Test Driven Development) you will write a test **before** writing the actual implementation for that piece of logic.

To demonstrate this, we will assume the interviewer gave us a simple FizzBuzz problem and he asked us to solve it using this [online java compiler](https://www.interviewbit.com/online-java-compiler/) for validating our output.
The requirements are simple: You have to iterate through a list of numbers from 1 to 100 and, for each of them, print "Fizz" if the number is a multiple of 3, "Buzz" if the number it s a multiple of 5, "FizzBuzz" if the number is a multiple of both and the number itself if none of the previous conditions were met.

The first instinct would be to start coding everything inside of the for loop.
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
But this piece of code will be very hard to test because all the output printed in the console. We can change our approach and extract our logic into a separate method, such as in the example below:
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
At this point you can already write your first 'tests'. 
Of Course we are not going to write actual junit tests, we will just call the getOutput() method with some specific parameters and check what it is returning. 
You can do it like this:
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

Now you can continue and finish the implementation. After this, the interviewer wll probably add new requirements such as this one:
"the program should print 'Woof' for dividers of 7". What will you do next? 

(Answer): write a test! 
```
assert getOutput(7).equals("Woof");
```
Sometimes the interviewer may leave some small 'holes' in the requirements to see if you manage to find and address them. At this point you should already ask what should the output be for input values such as 21 - should it be "Fizz"? "Woof"? "FizzWoof"? 
Always try find the areas where the requirement is unclear, ask them and keep track of the answers with... some new tests! 

```
assert getOutput(3*7).equals("FizzWoof");
assert getOutput(5*7).equals("BuzzWoof");
assert getOutput(3*5*7).equals("FizzBuzzWoof");
```

At this point you've got all the answers and you can continue coding.


## Why would you do this?
- it shows that your code is decoupled and testable
- it will be easier to check the output for certain values
- if you break something when implementing new requirements, you will detect it sooner
- you'll notice the missing requirements easier  

## Bonus - my implementation

Since I wrote this article and gave you this example, I have decided to give it a shot myself and wirte my own implementation of it.
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
