---
keywords: software
description: "It is common to see inheritance pattern in Python, but misusing so could lead to ill-maintained code when it scales. This post highlights a few situations in flavor of using inheritance pattern."
title: "Notes on \"Clean Code in Python\" — When to Apply Inheritance?"
toc: false
badges: true
comments: true
categories: [software, design]
image: images/2022-06-12-WhenInheritance/cover.jpg
layout: post
---

## **Motivation**
---

Inheritance is a pattern typically seen in [Object Oriented Programming (OOP)](https://en.wikipedia.org/wiki/Object-oriented_programming) languages such as Python, but you may see some articles (like [this one](https://codeburst.io/inheritance-is-evil-stop-using-it-6c4f1caf5117)) criticizing it.

This post continues to summarize my take-aways from ["Clean Code in Python"](https://www.amazon.com/Clean-Code-Python-maintainable-efficient/dp/1800560214). In the book, the author explains the trade-off for using inheritance, and highlight a few scenarios appropriate for applying inheritance.

## **Trade-off for Using Inheritance**
---

While inheritance has its own benefit, we should be mindful of the trade-off for using it.

### **✅ PRO: Reduce Code Repetition**

Inheritance reduces code duplication because any child classes could reuse the methods from its parent class. It is in line with [DRY (Don’t Repeat Yourself)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) principle.

A code with minimal repetition is readable as it avoids redundant information showing up over and over again. It is also easier to maintain as you only need to do code change in one place.

### **❌ CON: Higher Coupling**

Having said that, such benefit comes at a price. Inheritance introduces dependency between parent class and its subclass. Such interdependence is called [Coupling](https://en.wikipedia.org/wiki/Coupling_(computer_programming)).

Such dependency makes the code harder to maintain because any change you made in one of these classes inevitably propagates to its dependent classes. Such propagation is called [Ripple Effect](https://en.wikipedia.org/wiki/Ripple_effect).

When you have a gigantic hierarchy in your inheritance, even a tiny change in one class could bring unanticipated impacts on other modules.

### **❌ CON: Lower Cohesion**

Reusing methods from parent class sounds great! But what if you only need a small subset of them?

Inheritance renders the rest of the methods redundant, as if they should not belong to the class.

It signals your subclass has issue in terms of [Cohension](https://en.wikipedia.org/wiki/Cohesion_(computer_science)). It brings you technical debt because you have to take care of those unwanted methods. The situation is worse when some of the unwanted methods are public interfaces to the user. You may be aware which methods are unwanted, but your users are free to use any of them. So you have to deal it!

You gain code reusability, but you pay additional cost for maintaining these unwanted interfaces.

## **When NOT to Use Inheritance?**
---

_DON'T apply inheritance simply for the sake of reusing codes!_

Just because we get a few magic methods from a base class is not justified to introduce an inheritance. Don’t overlook the higher coupling and lower cohesion that it adversely introduces, it could easily outweigh the benefit. 

_DON'T apply inheritance when two objects are under “has-a” relationship!_

For example, a company has departments and employees. Department object shouldn’t inherit company object, so does employee object. A better alternative to represent such relationship is [Object Composition](https://en.wikipedia.org/wiki/Object_composition).

## **When to Use Inheritance?**
---

Inheritance should describe a "is-a" relationship. Child class should be functionally the same as its parent. Child class is a variant of its parent class. In addition, child class should serve as a specialization. It extends or modify features from its parents to serve a specific domain.

Below I summarise 3 scenarios appropriate for inheritance that the book showcases. For each scenario I attach a few examples from open source code.

## **Scenario 1**
---

Your parent class has captured the overall pipeline, but a few of its components depends on interfaces to be defined in child classes. It is easier to illustrate this by examples.

### **✨ Example: `BaseHTTPRequestHandler` and `SimpleHTTPRequestHandler`**

First example is extracted from the built-in [http] library. There is a [module](https://github.com/python/cpython/tree/3.10/Lib/http) that contains helper functions for server handling.

In the module, `BaseHTTPRequestHandler` is a class for handling HTTP requests in a server. This class implements a number of methods that can run independently. For example:

- `parse_request` for parsing a request
- `log_request` for logging an accepted request
- `send_header` for sending a header to buffer

But here notice how `handle_one_request` is defined. Pay attention to the middle:

```python
class BaseHTTPRequestHandler(socketserver.StreamRequestHandler):
    ####################################
    ### omit the rest of the methods ###
    ####################################
	
    def handle_one_request(self):
        try:
            self.raw_requestline = self.rfile.readline(65537)
            if len(self.raw_requestline) > 65536:
                self.requestline = ''
                self.request_version = ''
                self.command = ''
                self.send_error(HTTPStatus.REQUEST_URI_TOO_LONG)
                return
            if not self.raw_requestline:
                self.close_connection = True
                return
            if not self.parse_request():
                # An error code has been sent, just exit
                return
            mname = 'do_' + self.command
            if not hasattr(self, mname):
                self.send_error(
                    HTTPStatus.NOT_IMPLEMENTED,
                    "Unsupported method (%r)" % self.command)
                return
            method = getattr(self, mname)
            method()
            self.wfile.flush() #actually send the response if not already done.
        except TimeoutError as e:
            #a read or a write timed out.  Discard this connection
            self.log_error("Request timed out: %r", e)
            self.close_connection = True
            return
```

It attempts to grab and then call the target method (i.e. `method`) whose name has a pattern of “*do_<request type>”* (examples of request type are GET and POST).

However, it doesn’t provide any of these methods. In essence, `BaseHTTPRequestHandler` is designed to follow inheritance. Users are meant to define those interfaces in its child class, such as `SimpleHTTPRequestHandler`.

`SimpleHTTPRequestHandler` is also a HTTP request handler and intends to handle GET and HEAD request types with a simple rule. So it is a good choice to make `SimpleHTTPRequestHandler` extend from `BaseHTTPRequestHandler`:

```python
class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):
    ####################################
    ### omit the rest of the methods ###
    ####################################

    def do_GET(self):
        """Serve a GET request."""
        f = self.send_head()
        if f:
            try:
                self.copyfile(f, self.wfile)
            finally:
                f.close()

    def do_HEAD(self):
        """Serve a HEAD request."""
        f = self.send_head()
        if f:
            f.close()
```

### **✨ Example: `Callback` and `FetchPredsCallback`**

Another example is extracted from [fastai](https://github.com/fastai/fastai) library, a high level Deep Learning framework built on top of [PyTorch](https://pytorch.org/).

It has an interesting concept called Callback — an interface for user to flexibly intercept any phrase in a training loop and then inject customized procedures. It provide a list of phrases where you can intercept. For example, you can intercept at the beginning of training loop, or you can intercept the end of loss computation. You can read [this documentation](https://docs.fast.ai/callback.core.html) to learn more about it.

As the name suggests, `Callback` class (from this [module](https://github.com/fastai/fastai/blob/master/fastai/callback/core.py#L48)) is responsible for such callback interface.

Notice a short excerpt of its implementation. Pay attention to the middle:

```python
class Callback(Stateful,GetAttr):
    ####################################
    ### omit the rest of the methods ###
    ####################################

    def __call__(self, event_name):
        "Call `self.{event_name}` if it's defined"
        _run = (event_name not in _inner_loop or (self.run_train and getattr(self, 'training', True)) or
               (self.run_valid and not getattr(self, 'training', False)))
        res = None
        if self.run and _run:
            try: res = getattr(self, event_name, noop)()
            except (CancelBatchException, CancelBackwardException, CancelEpochException, CancelFitException, CancelStepException, CancelTrainException, CancelValidException): raise
            except Exception as e:
                e.args = [f'Exception occured in `{self.__class__.__name__}` when calling event `{event_name}`:\n\t{e.args[0]}']
                raise
        if event_name=='after_fit': self.run=True #Reset self.run to True at each end of fit
        return res
```

Its `__call__` method attempts to seek and then call the target method whose name is the value of `event_name`. The value of `event_name` representats the phrase that you want to intercept. For example, `before_epoch` represents the beginning of training loop and `after_loss` represents the end of loss computation.

However, `Callback` doesn’t have any of these methods provided. So `Callback` is  meant to be inherited!

One example of its child class is `FetchPredsCallback` -- a callback specialized in storing model prediction of validation sets. The step is done at the end of validation stage, suggested by its method name `after_validate`:

```python
class FetchPredsCallback(Callback):
    ####################################
    ### omit the rest of the methods ###
    ####################################

    def after_validate(self):
        "Fetch predictions from `Learner` without `self.cbs` and `remove_on_fetch` callbacks"
        to_rm = L(cb for cb in self.learn.cbs if getattr(cb, 'remove_on_fetch', False))
        with self.learn.removed_cbs(to_rm + self.cbs) as learn:
            self.preds = learn.get_preds(ds_idx=self.ds_idx, dl=self.dl,
                with_input=self.with_input, with_decoded=self.with_decoded, inner=True, reorder=self.
```

## **Scenario 2**
---

You want to enforce the same interfaces across class objects. You can make use of parent class as an abstract class.

An abstract class enforces a contract with its child classes. It declares interfaces without a need to implement them, but its child classes must implement them in order to be instantiated.

### **✨ Example: `abc` Module**

`abc` is a handy library to help you define abstract class.

Any class inherited from `abc.ABC` class is treated as an abstract class and can’t be instantiated. You can declare the “contract-binding” interfaces with `abc.abstractmethod`. See its [documentation](https://docs.python.org/3/library/abc.html) for more details.

Here I provide a simple example on how to use `abc` module to define abstract class:

```python
import numpy as np
from abc import ABC, abstractmethod
from sklearn.linear_model import LinearRegression 

class ABCModel(ABC):
    @abstractmethod
    def train(self, X: np.ndarray, y: np.ndarray):
        ...

    @abstractmethod
    def predict(self, X: np.ndarray) -> np.ndarray:
        ...

class LinearModel(ABCModel):
    def __init__(self):
        self._model = LinearRegression()

    def train(self, X, y):
        self._model.fit(X, y)

    def predict(self, X):
        return self._model.predict(X)
```

### **✨ Example: `Dataset` and `CIFAR10`**

Another example that fits into this category is torchvision’s `Dataset` class.

PyTorch has its own pipeline to do data loading. To leverage the pipeline it built, you must implement the step to fetch samples from a data set. It’s like filling up a missing piece to complete a puzzle. Once it’s filled, the fetch samples can be shuffled and batched by `Dataloader` class to serve a neural network model.

`Dataset` is the abstract class that enforces this constraint. User has to inherit from this class and implement `__getindex__` method for indexing a sample from a data set: 

```python
class Dataset(Generic[T_co]):
    ####################################
    ### omit the rest of the methods ###
    ####################################

    def __getitem__(self, index) -> T_co:
        raise NotImplementedError
```

Notice `Dataset` doesn’t make use of `abc` module, so its “contract-binding” interface `__getitem__` has to add a line: `raise NotImplementedError`.

`CIFAR10` is an example of its child class. It represents [CIFAR10](http://www.cs.toronto.edu/~kriz/cifar.html) dataset — a bechmark dataset for classification task in computer vision. See how `CIFAR10` implements `__getitem__` to fetch an image and its associated label from the data set.

```python
class CIFAR10(VisionDataset):
    ####################################
    ### omit the rest of the methods ###
    ####################################

    def __getitem__(self, index: int) -> Tuple[Any, Any]:

        img, target = self.data[index], self.targets[index]

        # doing this so that it is consistent with all other datasets
        # to return a PIL Image
        img = Image.fromarray(img)

        if self.transform is not None:
            img = self.transform(img)

        if self.target_transform is not None:
            target = self.target_transform(target)

        return img, target
```

Note that `VisionDataset` stems from `Dataset` abstract class so any inheriting class follows the same contract.

## **Scenario 3**
---

The third scenario suitable for inheritance is exception handling.

You can segregate a more generic error into more specific error with the help of inheritance.On one hand, a child error could handle a specific type of error. It better helps developers identify the root cause of a failure. On the other hand, it remains the flexibility to fall back to its generic “parent error”.

### **✨ Example: `ContentTooShortError` and `URLError`**

A example I took here is referenced from the builtin [urllib](https://github.com/python/cpython/tree/3.10/Lib/urllib) library. It defines a number of errors customized to URL handling.

`URLError` indicates a generic failure caused during accessing a URL link. Such failure could have many possible causes — invalid URL, incomprehensible status code from the server response... etc. `ContentTooShortError` represents one of the cause: the downloaded file from the URL is incomplete, meaning the file size smaller than expected.

To reflect such hierarchy, `ContentTooShortError` inherits from `URLError`:

```python
class ContentTooShortError(URLError):
    """Exception raised when downloaded size does not match content-length."""
    def __init__(self, message, content):
        URLError.__init__(self, message)
        self.content = content
```

Here is a sample code to show the hierarchy. Thanks to the construction of `ContentTooShortError`, we can catch and handle this specific error if wanted. Otherwise, we could fallback to handle it like a more generic error, such as `URLError`. 

```python
try:
    raise ContentTooShortError("content too short", "") 
except ContentTooShortError:
    print("Caught by ContentTooShortError exception") 
except URLError:
    print("Caught by URLError exception") 
except:
    print("Caught by other exception")
# >>"Caught by ContentTooShortError exception"


try:
    raise ContentTooShortError("content too short", "")
except URLError:
    print("Caught by URLError exception")
except:
    print("Caught by other exception")
# >>"Caught by URLError exception"
```

## **References**
---
1. [Clean Code in Python: Develop maintainable and efficient code, 2nd Edition](https://www.amazon.com/Clean-Code-Python-maintainable-efficient/dp/1800560214)
2. [Inheritance and Composition: A Python OOP Guide](https://realpython.com/inheritance-composition-python/)
3. [When to Use Inheritance and When Not to in OOP?](https://python.plainenglish.io/when-to-use-inheritance-and-when-not-to-in-oop-1cac5d4f049)
4. [Inheritance Is Evil. Stop Using It.](https://codeburst.io/inheritance-is-evil-stop-using-it-6c4f1caf5117)
5. [FastAI Documentation: Callbacks](https://docs.fast.ai/callback.core.html)