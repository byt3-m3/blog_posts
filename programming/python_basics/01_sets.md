# Introduction into Python Sets 

In this blog we will discuss probably one of the least utilized data structures available in python among network engineers.  As i read through code from peers and other techies, I personally do not see sets being implemented all that much. The concepts for sets is pretty straightforward, a Set is a collection of elements that do not contain duplicates.  Now as simple as that sound lets see where this data structure can be applicable for us Network folks. Ever had to pull a list of assets from a management system and that system has redundant information such as multiple hostnames associated with duplicated management IPs.  Instead of writing a bunch of noisy normalization code that parses and reorganize the data, let python do it for you. 

## Implementation

First lets say we received a file called "hosts.csv" that was exported from  NMS. This export contains  the hostname in Column 1 and the ip addresses in Column 2. 

hosts.csv

```reStructuredText
r1,192.168.1.1
r2,192.168.1.2
r3,192.168.1.3
r3_1,192.168.1.3
r4,192.168.1.4
r5,192.168.1.5
```

Next lets write some code to open the file and extract its content. 

```python
import csv


def w_set():
    with open("hosts.csv", 'r') as file:
        csv_file = csv.reader(file)
        ip_set = {row[1] for row in csv_file} # Set comprehensions 

    print(len(ip_set))


if __name__ == "__main__":
    w_set()
```

 We used the csv package in the standard lib to accomplish this.   We will use a set comprehension to iterate over csv_file content and extracts all non-duplicate IPs. At the end of the function we will print the length of the Set. now lets run it 

output 

```reStructuredText
>>> 5
```

You will notice that the length of the ip_set variable == 5, yet the csv file only contained 6 elements. At this point I would assume you know the answer to that but as stated before sets will contain no duplicates.  "192.168.1.3" appears twice in this file and the Set comprehension was able to determine this without any additional helper code. Ill give you an example of how I see problems like this addressed without sets. 

```python
import csv


def no_set():
    ip_list = []
    with open("hosts.csv", 'r') as file:
        csv_file = csv.reader(file)
        for row in csv_file:
            if row[1] not in ip_list:
                ip_list.append(row[1])

    print(len(ip_list))


if __name__ == "__main__":
    main()
```

output

```reStructuredText
>>> 5
```

There is nothing really wrong with this code but IMO its not beautiful and its hard to read without knowing the context of the requirement. 

## Conclusion

 Python provides us some pretty great built-in functionality and encourage all devs new and old to review the online python standard docs. Every time I review a section in the docs I always end up walking away with something new. 