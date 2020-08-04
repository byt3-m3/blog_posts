# Behavioral Design Patterns: Command Pattern

 

## Overview

Have you ever wanted to write a SSH robot that takes a sequence of commands  and executes each of them. Or did you ever consider creating a Self-Healing tool that will reactivley send predefined commands to an asset remmidatating the detected issue.  The command is powerful pattern that allows you abstract the action of the command itself. This is accomplished by way of Interfaces. Concrete Implementation of the Command class will focus on indecent implmenation. In this blog we will be impmenting  simple Command pattern. 



### Design

In this example it is assumed that you understand the following technologies and concepts

- Python3
  - Classes
  - Interfaces 

**Requirements**

- Execute System Commands using CLI
- add ability to view the call history 

For this project we will be building a CLI tool that can execute any system command. The engineers that will be consuming this service request that module provides them information regarding how many times the Command object was called and wheter the execution was seccesful or not. 



## Implementation 

First we will need define our ICommand interface. If you are not familiar with Interfaces in python, its ok, just know, Interfaces are  essentially blueprints for classes. In static type langauges they aid in a concept call polymorphism which is huge  topic on its own, for now just know in order to us Interfaces in python you must use the abc package.

icommand.py

```python
class ICommand(ABC):

    @abstractmethod
    def execute(self) -> CommandResult:
        raise NotImplementedError
```



As you can see the execute() method is define but no logic is written. This has to do with the Interface concept, Im not going to go into the details of excalty how this works, just know that any class that inherits rom the ICommand interface  will be forced to do their own Implmentation of the execute() method. 



Next we will Create a BaseCommand object that will use ICommand as it's base class. 

```python
class BaseCommand(ICommand):
    def __init__(self, **kwargs):
        self.cmd = kwargs.get("cmd")
        self.arg = kwargs.get("arg")
        self._run_log = []
        self._runs = 0
				
    def execute(self):
        raise NotImplementedError

    def add_result(self, result: CommandResult) -> bool:
      	if isinstance(result, CommandResult):
          self.results = result 
    
          return True
        else:
          return False
    
   def del_result(self, result: CommandResult) -> bool:
      	if isinstance(result, CommandResult):
          if result not in self.results:
            return False
          else:
           self.results.remove(result)
           return True
          
   def clear_results():
    	self._run_log = []
      
   @property
  	def calls(self):
      return len(self._run_log)

    @property
    def results(self):
        return self._run_log

    @results.setter
    def results(self, result: CommandResult):
        if isinstance(result, CommandResult):
            self._run_log.append(result)

    def __repr__(self):
        return f"<Command(cmd='{self.cmd}', arg='{self.arg}')>calls='{self.calls}'"
```

As you can see the BaseCommand class inherits from the ICommand class. This forces the BaseCommand class to implement the execute() method. As you can tell there is no logic and that is  due to the fact that this class is not meant to be a concrete implementation of a command.  For specilzed Commands, They will inherit from the BaseClass opposed to the ICommand interface. Take note of a few properties that we declared for the class, The 'calls' property will return a int representation of how many times the Command object was called.  The 'results' property will return a list of CommandResult(Not Yet Defined) objects. This calls will be created at a later time. The add_result, del_result, and clear_result methods should be self expalintory but this will provide the users basic functionlaity when using building classes using our BaseCommand class. 



Next We need to define the CommandResult class that we have seen some many time thus far. When I ever I use this pattern I tend to make my return types consistent from a consumer point of view. This allows the consumer to be able to predict the return type everytime they call the execute() method. Type checking can be preforomed on the CommandResult.type attribute by the consumer allowing more flexibilty and easiblity when using my command classes.  When the class is create it will take 2 params, Data, which will be data you want to store, Status, which repsents wiether the execution was successful or not. The type attrib will automtically be assigned as it will get the type value for the data attrib. 

```python
class CommandResult:
    def __init__(self, data, status: bool = False):
        self.data = data
        self.status = status
        self.type = type(self.data)

    def __repr__(self):
        return f'<CommandResult(status={self.status}, type={self.type})>'
```



Next based on our requirements I think I will be using the os.popen function to execute system calls. Because an OS has allot of possible commands and can take various amount of arguments  we should define a concrete class enfocing these constraints 



 ```python
import os 

class OSCommand(Command):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)

    def execute(self):
  
        res = os.popen(f'{self.cmd} {self.arg}').read()
    		c_result = CommandResult(res, True)
        self.add_result(c_result)
        return c_result
 ```



Next use this class as if as consumer would utlize it. 

```python
def main():
    ping = OSCommand(cmd="ping", arg='8.8.8.8')
    nslookup = OSCommand(cmd="nslookup", arg='google.com')
    arp = OSCommand(cmd="arp", arg='-a')
    
    ping.execute()
    ping.arg = '1.1.1.1'
    pring.execute()
    nslookup.execute()
    arp.execute()
```



