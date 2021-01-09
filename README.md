<p align="center">
  <img src="https://github.com/chriskiehl/re-wx-images/raw/images/logo/rewx.png"> 
</p>

<p align="center">A Python library for building modern declarative desktop applications</p>

<hr/>

![PyPI](https://img.shields.io/pypi/v/gooey)


# Overview

re-wx is a library for building modern declarative desktop applications. It's built as a management layer on top of WXPython, which means you get all the goodness of a mature, native, cross-platform UI kit, wrapped up in a modern, React inspired API. 

## What is it? 

It's a "virtualdom" for WX. You tell re-wx what you want to happen, and it'll do all the heavy lifting required to get WX to comply. It lets you focus on your state, transitions, and business logic while leaving implentation details of WX's ancient API to re-wx.  

**Say goodbye to** 

* Deep coupling of business logic to stateful widgets
* Awkward auto-generated Python wrappers on old bloated C++ classes 
* Being forced to express UIs through low level `A.GetLayout().addChild(B)` style plumbing code 

Re-wx lets you build expressive, maintainable applications out of simple, testable, functions and components.

## Installation 

The latest stable version is available on PyPi. 
```
pip install rewx 
```

## Documentation

* Tutorial: Intro to re-wx 
* Main Concepts 
* Advanced Concepts
* Debugging 



<h2 align="center">RE-WX in Action!</h2>


### A tiny Hello World

<img src="https://github.com/chriskiehl/re-wx-images/raw/images/screenshots/hello-world.png" align=right >
You can assemble applications with re-wx using the humble function. This takes data and returns data. re-wx handles all the lifting required to build the WX instances. 



```python
import wx
from rewx import create_element, wsx, render
from rewx.components import StaticText, Frame


def say_hello(props):
    return wsx(
        [Frame, {'title': 'My first re-wx app', 'show': True},
         [StaticText, {'label': f'Hello, {props["name"]}!'}]]
    )


if __name__ == '__main__':
    app = wx.App()
    frame = render(create_element(say_hello, {'name': 'cool person'}), None)
    app.MainLoop()
```

<br/><br/>
### A Stateful component 

<img src="https://github.com/chriskiehl/re-wx-images/raw/images/screenshots/clock.png" align=right >

Components allow you to store and manage state. Behind the scenes, re-wx handles the details of making WX's widgets match your applications state. 

```python
class Clock(Component):
    def __init__(self, props):
        super().__init__(props)
        self.timer = None
        self.state = {
            'time': datetime.datetime.now()
        }

    def component_did_mount(self):
        self.timer = wx.Timer()
        self.timer.Notify = self.update_clock
        self.timer.Start(milliseconds=1000)

    def update_clock(self):
        self.set_state({'time': datetime.datetime.now()})

    def render(self):
        return wsx(
          [c.Block, {},
           [c.StaticText, {'label': self.state['time'].strftime('%I:%M:%S'),
                           'name': 'ClockFace',
                           'foreground_color': '#51acebff',
                           'font': big_ol_font(),
                           'proporton': 1,
                           'flag': wx.CENTER | wx.ALL,
                           'border': 60}]]
        )
```


<br/><br/>
### An Application

<img src="https://github.com/chriskiehl/re-wx-images/raw/images/screenshots/todo-app.png" align=right >

```python 
def TodoList(props):
    return create_element(c.Block, {}, children=[
        create_element(c.StaticText, {'label': f" * {item}"})
        for item in props['items']
    ])


class TodoApp(Component):
    def __init__(self, props):
        super().__init__(props)
        self.state = {'items': ['Groceries', 'Laundry'], 'text': ''}

    def handle_change(self, event):
        self.set_state({**self.state, 'text': event.String})

    def handle_submit(self, event):
        self.set_state({
            'text': '',
            'items': [*self.state['items'], self.state['text']]
        })

    def render(self):
        return wsx(
            [c.Frame, {'title': 'My First TODO app'},
             [c.Block, {'name': 'main-content'},
              [c.StaticText, {'label': 'What needs to be done?'}],
              [c.TextCtrl, {'value': self.state['text']}],
              [c.Button, {'label': 'Add', 'on_click': self.handle_submit}],
              [c.StaticText, {'label': 'TO DO:'}],
              [TodoList, {'items': self.state['items'], 'on_click': self.handle_complete}]]]
        )

if __name__ == '__main__':
    app = wx.App()
    frame = render(create_element(TodoApp, {}), None)
    frame.Show()
    app.MainLoop()
```



Note that re-wx is _"just"_ a library, _not_ a framework. It 100% compatible with your existing WX codebase. You can use as much or as little of re-wx as you want. 








### Flexible

You can use as little or as much of re-wx's capabilities as you want. Meaning, if you don't buy into the state management or data oriented aspects, you can use re-wx purely as a more declarative way of managing layouts in WX, while still managing all other interactions the traditional way.  

```python
class MyPanel(wx.Panel): 
    def __init__(*args, **kwargs): 
        super().__init__(*args)
        self.input1 = rewx.Ref() 
        self.input2 = rewx.Ref() 
        self.button = rewx.Ref() 
        render(self.layout(), self) 
        
    def component_did_mount(self): 
        # Your layout has been built and 
        # all components instantiated. You can now 
        # proceed as usual. 
        self.input1.instance.Bind(wx.EVT_TEXT, self.my_handler1)
        self.input2.instance.Bind(wx.EVT_TEXT, self.my_handler2)
        # and so on       
        
    def layout(self): 
        return wsx(
            [StaticText, {'ref': self.input1}]
        )
        
        

```



Tutorial: 

The core of re-wx is the humble `Element`. It's just a piece of plain data that describes the type of component to create, what its properties should be, and if it should have any children. 

```
from rewx import create_element
from rewx import StaticText 

my_first_elelement = create_element(StaticText, {'label': 'Hello world!'})
```

You build UIs in re-wx by creating trees of these components. 

```
from rewx import create_element
from rewx import StaticText, Block  

a_tree_of_elements = create_element(Block, {}, children= [
    create_element(StaticText, {'label': 'I am the first child!'}),
    create_element(StaticText, {'label': 'I am the second!'})
])
```

You assemble UIs in re-wx by assembling trees of _Components_ and _Elements_. Components are functions or classes which  






```python
class FormControls(wx.Panel): 
    def __init__(*args, **kwargs): 
       super().__init__(*args, **kwargs)
       self.text_entry = wx.TextCtrl(self)
       self.button = wx.Button(self, label='Ok')
       self.button.Bind(wx.EVT_BUTTON, self.on_click)
       
       hsizer = wx.BoxSizer(wx.HORIZONTAL)
       hsizer.Add(self.text_entry, 1) 
       hsizer.Add(self.button, 0)
       vsizer = wx.BoxSizer(wx.VERTICAL) 
       vsizer.Add(hsizer, 1, wx.EXPAND)
       self.SetSizer(vsizer)
```

## with RE-WX 

```python
def form_controls(props): 
   return wsx(
     [Block, {'orient': wx.VERTICAL}, 
       [Block, {'orient': wx.HORIZONTAL}, 
         [StaticText, {'label': 'Foobar'}],
         [Button, {'label': 'Submit', 'on_click': props['on_click']}]]]
   )
```



 * Declarative: 
 * Component based -  
 * complete interop with the rest of your WX codebase - re-wx doesn't require you to change anything about your current codebase to start using it. Just create a component, attach it to an existing set of WX widgets, and off you go! 

Get away from managing the low level details of WX's individual widgets and components. You tell re-wx what you want your UI to do, and it'll do all the heavy lifting to get WX to comply.  

re-wx is an implementation of React's ideas _on top_ of WX Python. It allows you to decouple your state and logic from your UI, and declaratively state blah blah blah. 



```python

def controls(props): 
    return wsx(
        [Block, {'orient': wx.HORIZONTAL}, 
          [Button, {'on_click': props['on_start'], 'label': 'Ok'}],
          [Button, {'on_click': props['on_cancel'], 'label': 'Cancel'}]]
    )
```

### virtualframe and diffing 

### WSX

`create_element` is the fundamental building block of all of re-wx. However, it's a bit verbose and can make viewing your UI's structure at a glance difficult as it gets lost in a sea of method call noise. As an alternative, `wsx` lets you use nested lists to express parent child relationships between components. 

```python
def my_component(props): 
   return wsx(
     [Block, {'orient': wx.VERTICAL}, 
       [Block, {'orient': wx.HORIZONTAL}, 
         [StaticText, {'label': 'Foobar'}],
         [Button, {'label': 'Submit', 'on_click': props['on_click']}]]]
   )
```

the `wsx` call will transform the lists into an equivalent `create_element` form 

```python
def my_component(props): 
   return create_element(Block, {'orient': wx.VERTICAL}, children=[
       create_element(Block, {'orient': wx.HORIZONTAL}, children=[
           create_element(StaticText, {'label': 'Foobar'})
           create_element(StaticText, {'label': 'Submit', 'on_click': props['on_click']})
       ])
   ])
```

Two paths to the same place, so you can use whichever one feels most natural. 



WX Widgets currently managed by re-wx. 



Head on over to the Getting Started guide to find out more. 

Rationale: 

Development in WX blows.Wrappers around C++ classes. Evertything requires subclassing a low-level plumbing code 




## Philosophy

**It's a library first.**
re-wx is "just" a library. it is _not_ a framework. You can use as much or as little of it as you need. Further, the output from a re-wx `render` is a plain old WXPython component. Meaning, all re-wx components _ARE_ WX components, and thus require no special handling to integrate with your existing code base. 

**It is not trying to hide WXPython** 
re-wx is not trying to be an general purpose abstraction over multiple backend UI kits. It's lofty goals begin and end with it being a way of making writing native, cross-platform UIs in WXPython easier. As such, it doesn't need reconcilers, or generic transactions, or any other bloat. re-wx entire codebase is just a handful of files < 1k lines of code, and could be understood in an afternoon.  

As such, practicality is favored over purity of abstraction. Meaning, you'll mix-match WXPython code + re-wx code as needed. A good example of this is for transient dialogs (confirming actions, getting user selectsions, etc..). In React land, you'd traditionally have a modal in your core markup, and then conditionally toggle its visibility via state. However, in re-wx, you'll just use the dialog directly rather than embedding it in the markup and handling its lifecycle via `is_open` style state flags. This is practical to do because, unlike React in Javascript, WX handles managing the UI thread thus allowing us to block in place without any negative effects. Which enables writing straight forward in-line Dialog code.  

```python
def handle_choose_dir(self, event): 
    dlg = wx.DirDialog(None)
    if dlg.Show() == wx.ID_OK:
        self.setState({'directory': dlg.GetPath()})
``` 

Events: re-wx does no event wrapping. The normal WX events are used. 

It's not trying to be an opaque framework hiding away the internal details of WX. It's very much designed to operate on-top of and in concert with WX. 

## Compromises and caveats in the design

It's not a true one-way data flow for certain components. For instance, in WX, ComboBoxes only produce events _after_ it's internally updated its state. 

Another example is TextCtrl. EVT_TEXT is fired _after_ the textctrl has been updated. There's EVT_CHAR, however, it's so low level that handling it would require essentailly reimplementing the TextCtrl itself from scratch. 

However, in practice, this tends to not matter much. You can still update the control in rsponse to the event, which generally happens fast enough that the transient state isn't noticed by the user (if shown on the screen at all!)


More caveats: 

Only the most common attributes are currently managed by declarative props (basically, most of what falls under `wx.Control`). Specifics such as `InsertionPoint`s in TextCtrls are considered out of scope for rewx. `Ref`s act as a handy escape-hatch for any lower level API needs .

More Caveats: 

The prefab RadioGroup cannot have its number of options changed after creation. So, jiggling the `choices` will have no effect. The good news is that RadioGroup is largely just a convenience class, if you need to dynamically vary the options, you can simulate the RadioGroup via 

TODO
```python
class MyRadioGroup(Component): 
    def render(self):
        return wsx(
            ['borderbox', {} ]
        )


``` 



# Current state of the world: 

Implementation notes: 

Reconciliation doesn't have many performance focused smarts at this point. It only looks at the current props to decide what to do, and thus blanket unsets/resets values every time, as opposed to a more nuanced approach which diff'd the prev/current props and conditionally updated only those pieces which need changes. However, so far, with moderately sized UIs, this naieve approach hasn't produced any notable performance issues. This may change in the future.  
 


# Important Note!

Lambdas inside of lists: 

```python
[{
    'attrs': {
        'on_click': lambda x: self.doThing(dynamicThing) 
    }
} for dynamicThing in self.state['things']]
```

This is wrong! See: [link to python docs ]

Use `callwith`
```python
[{
    'attrs': {
        'on_click': callwith(self.doThing, dynamicThing) 
    }
} for dynamicThing in self.state['things']]

```

## Stuck? Need some help? Just have a question? 

Open an issue [here]()
Or feel free to hit me up at me@chriskiehl.com


## Contributing

All contributions are welcome! Just make sure you follow the Contributing Guidelines. 


## License

re-wx is MIT licensed.
