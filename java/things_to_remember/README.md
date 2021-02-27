# Things to remember

Note pad for things I commonly forget in Java.

## Arrays

### Common methods

```java
// declaration
int[] myNums = {10, 20, 30, 40};
int[] myNumsMoreVerbose = new int[]{10, 20, 30, 40};
int[] myNumsOfSize = new int[4]; // 4 elements
int [] myNumsFromStream = IntStream.range(0, 100).toArray(); // From 0 to 99
String[] myCars = {"Volvo", "BMW", "Ford", "Mazda"};

// length
int myNumLength = myNums.length;

// printing
System.out.println(Arrays.toString(myNums););


// create from Arraylist
    ArrayList<String> arrayList = new ArrayList<>(Arrays.asList(myCars));
// N.B. There is no shortcut for making the conversion if the type of the Array is primitive (a for loop would be needed)

// reverse an array
ArrayUtils.reverse(intArray);

// remove an element
int[] removed = ArrayUtils.removeElement(myNums, 30); //creates a new array!

// check if a certain value exists
boolean b = Arrays.asList(myCars).contains("a"); // no need to do full declaration of conversion to ArrayList for successful compilation




```

### Comparison with Collections

| Topic |	Arrays |	Collection |
| ------------- | ------------- | ------------- |
|Size |	Fixed	 | Dynamic
| Memory Consumption |	Possibly more due to fixed size | Possibly less due to dynamic size
| Data type |	Only raw types, NO generics |	Generics allowed
| Primitives storage | 	Both object and primitive type data. |	Only object, not primitive
| Performance |	Better |	Worse

## Bytes