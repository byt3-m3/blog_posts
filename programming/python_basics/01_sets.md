# Introduction into Python Sets 

This blog will discuss one of the least utilized data structures available in python among network engineers. As I read through code from peers and other techies, I personally do not see Sets being implemented all that much. The concepts for Sets is pretty straightforward, a Set is a collection of elements that do not contain duplicates. As simple as that sound, let's see where this data structure can be applicable for us Network folks. 

Ever had to pull a list of assets from a management system. That system has redundant information, such as multiple hostnames associated with duplicated management IPs. Instead of writing a bunch of noisy normalization code that parses and reorganize the data, let python do it for you.

## Implementation

First, let's say we received a file called "hosts.csv" that was exported from NMS. This export contains the hostname in Column 1 and the IP addresses in Column 2.

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
        ip_set = {row[1] for row in csv_file}

    print(len(ip_set))


if __name__ == "__main__":
    w_set()
```

We used the CSV package in the standard lib to accomplish this. We will use a set comprehension to iterate over csv_file content and extracts all non-duplicate IPs. At the end of the function, we will print the length of the Set. now lets run it

output 

```reStructuredText
>>> 5
```

You will notice that the length of the *ip_set* variable == 5, yet the CSV file only contained 6 elements. I would assume you know the answer to that, but as stated before, sets will contain no duplicates. "192.168.1.3" appears twice in this file, and the Set comprehension was able to determine this without any additional helper code. I'll give you an example of how I see problems like this addressed without sets. 

```python
import csv


def wo_set():
    ip_list = []
    with open("hosts.csv", 'r') as file:
        csv_file = csv.reader(file)
        for row in csv_file:
            if row[1] not in ip_list:
                ip_list.append(row[1])

    print(len(ip_list))


if __name__ == "__main__":
    wo_set()
```

output

```reStructuredText
>>> 5
```

There is nothing really wrong with this code, but IMO is not beautiful, and it's hard to read without knowing the context of the requirement. 

## Conclusion

Python provides us some pretty great built-in functionality and encourages all devs new and old to review the online python standard docs. Every time I check a section in the docs, I always end up walking away with something new.