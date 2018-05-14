---
title: "The Evolution of the Many Faced for Loop"
date: 2017-12-03T21:52:01-05:00
categories: [Programming]
tags: ["loops"]
draft: true
---

### What are loops?
In it's simplest form, a loop is a an action that's being repeated over a period of time or given a specific condition. Everyday, when i wake up, if running water is available, i brush my teeth then i take a shower. That was an example of a loop where i perform two actions **every day** given that there is water available.


### The For loops
In computer programming, there are many ways to represent a loop and it has evolved over the years with the re-introduction of the functional programming paradigm and the explosion of different programming languages.

The most basic of loops is the **for** loop. If i represent the given example in the introduction paragraph in different programming languages, we can closely look at the differences and similarities. To make these examples more diverse, i will use a mix of static and dynamic languages: `java, go, python and javascript`, with the following assumptions:

- The age where someone can brush their teeth and shower by themselves is 6 years old. We will represent the age in days as 6 * 365 days, since the we brush our teeth daily not yearly.
- Life expectancy is 82 years, we declare that as No of days in a variable as 82 * 365.

it would look like so:

#### Java v1.8.0_131

```Java
public static void main(String[] args) {
        int lifeExpectancy = 82 * 365;
        int independentAge = 6 * 365 ;
        int totalTimeIndependent = lifeExpectancy - independentAge
        for (int i = 0; i < totalTimeIndependent i++) { //
            BrushTeeth(i);
            Shower(i);
        }

    }
    public static void BrushTeeth() {
        System.out.println("Currently brushing my teeth...");
        System.out.println("Done brushing my teeth...");
    }

    public static void Shower() {
        System.out.println("Currently showering...");
        System.out.println("Done Showering...");

    }
```
### Go v1.10
```go

func main() {
	lifeExpectancy := 82 * 365
	independentAge := 6 * 365
	totalTimeIndependent := lifeExpectancy - independentAge
	for i := 0; i < totalTimeIndependent; i++ {
		brushTeeth(i)
		shower(i)
		
	}

}
func brushTeeth(day int) {
	fmt.Println("Currently brushing my teeth on day %n...", day)
	fmt.Println("Done brushing my teeth")
}
func shower(day int) {
	fmt.Println("Currently showering on day %n ....", day)
	fmt.Println("Done showering")
}

```
### Python v2.7.10

```python
def main():
    lifeExpectancy = 82 * 365
    independentAge = 6 * 365
    totalTimeIndependent = lifeExpectancy -independentAge
    for x in range(totalTimeIndependent):
        brushTeet(x)
        shower(x)
        
def brushTeet(day):
    print("Currently brushing my teeth on day %s..."%(day))
    print("Done brushing my teeth")
def shower(day):
    print("Currently showering on day %s..."%(day))
    print("Done showering")

if __name__ == '__main__':
    main()

```
### Javascript

```javascript
function init(){ 
    const lifeExpectancy = 82 * 365
	const independentAge = 6 * 365
	const totalTimeIndependent = lifeExpectancy - independentAge
	for (let i= 0; i < totalTimeIndependent; i++){
       brushTeeth()
       shower()
    }

}
function  brushTeeth(day) {
	console.log("Currently brushing my teeth on day %n...", day)
	console.log("Done brushing my teeth")
}
function  shower(day) {
	console.log("Currently showering on day %n ....", day)
	console.log("Done showering")
}

```

Notice how the syntax is clear and very similar between the 4 languages, well except that in python the `range` construct is used. Over the years, languages have borrowed from functional programming to make loops more powerful.

### The functional for loops



