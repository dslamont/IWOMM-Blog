+++
title = "Object Ids to CSV"
date = 2022-04-30T14:00:00+01:00
images = []
tags = ["solidcode"]
categories = ["solidcode"]
draft = false
slug=""
+++

A colleague at work asked me recently how to format a url's ```QueryString``` to specify multiple records. I suggested concatenating the record's ```Id``` values as a CSV (comma separated values) ```string``` such as ```?ids=1,2,3```. Then they asked how to create such a ```string```. I was slightly taken aback as it seemed a simple thing to do. 

I suggested they use ```Linq``` to extract the record ids then use  ```String.Join()``` to create the CSV string. Afterwards I started to wonder if this was the best option so I decided to investigate.

I wrote the following code to test whether using ```Linq``` and ```String.Join()``` was a better option than just using a simple loop.

``` csharp {linenos=false}
public class CSVTest
{
    private const int TEST_SIZE = 500;
    private readonly List<TestEntry> entries = new List<TestEntry>(TEST_SIZE);
    private readonly int[] idsArray = new int[TEST_SIZE];
    public CSVTest()
    {
        for (int i = 0; i < TEST_SIZE; i++)
        {
            TestEntry entry = new TestEntry();
            entry.Id = i;
            entry.Desc = $"Test Entry {i}";
            entries.Add(entry);

            idsArray[i] = i;
        }
    }

    [Benchmark]
    public string LoopTest()
    {
        StringBuilder output = new StringBuilder();

        for (int i = 0; i < entries.Count; i++)
        {
            if (i > 0)
            {
                output.Append(",");
            }
            output.Append(entries[i].Id);
        }

        return output.ToString();
    }

    [Benchmark]
    public string LinqStringJoinTest()
    {
        int[] ids = entries.Select(te => te.Id).ToArray();
        string output = String.Join(",", ids);

        return output;
    }

    [Benchmark]
    public string StringJoinTest()
    {
        return String.Join(",", idsArray);
    }

    public class TestEntry
    {
        public int Id { get; set; }
        public string Desc { get; set; }
    }
}
```

This produced the following results

```

|             Method |      Mean |     Error |    StdDev |
|------------------- |----------:|----------:|----------:|
|           LoopTest |  5.588 μs | 0.0895 μs | 0.0837 μs |
| LinqStringJoinTest | 10.407 μs | 0.0819 μs | 0.0726 μs |
|     StringJoinTest |  9.311 μs | 0.1226 μs | 0.1147 μs |
```

It can be seen that using ```String.Join()``` slower than using a loop and ```StringBuilder```. Initially I thought using ```Linq``` would be the bottleneck but it is ```String.Join()``` itself.

Will I just use loops in the future to create CSV strings. Probably not as ```Linq``` and ```String.Join()```  is a lot more succinct and readable and the time differences aren't too large in the great scheme of things. So unless there is a pressing need for performance then I wouldn't bother hand coding loops.

This investigation does highlight an important principle. Less lines of code does not necessarily lead to better performance.