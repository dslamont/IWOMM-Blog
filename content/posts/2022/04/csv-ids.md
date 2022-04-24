+++
title = "Object Ids to CSV"
date = 2022-04-16T11:15:00+01:00
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
    private readonly List<TestEntry> entries = new List<TestEntry>();

    public CSVTest()
    {
        for (int i = 0; i < 500; i++)
        {
            TestEntry entry = new TestEntry();
            entry.Id = i;
            entry.Desc = $"Test Entry {i}";
            entries.Add(entry);
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
|           LoopTest |  5.460 μs | 0.0616 μs | 0.0546 μs |
| LinqStringJoinTest | 10.438 μs | 0.0899 μs | 0.0797 μs |

```

It can be seen that using ```Linq``` and ```String.Join()``` was twice as slower. I suspect using ```Linq``` to create the ```array ``` of ids is the slow part. Will I just use loops in the future to create CSV strings. Probably not as ```Linq``` and ```String.Join()```  is a lot more succinct and readable and the time differences aren't too large in the great scheme of things. So unless there is a pressing need for performance then I wouldn't bother hand coding loops.

This investigation does highlight an important principle. Less lines of code does not necesarily leed to better performnce.