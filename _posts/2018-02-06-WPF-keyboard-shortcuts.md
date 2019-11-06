---
layout: post
title:  "Keyboard Accelerators (Shortcuts) in C# WPF (A minimal example)"
date:   2018-02-06 14:00:00 +0000
categories: c# wpf keyboard-accelerators
tags: c# wpf keyboard-accelerators
---

This is a simple example of how to implement a keyboard shortcut in C# WPF. I couldn't get a lot of code samples available out there to work so this is just a minimal example of one menu command.

[Link with repo with example source code](https://github.com/mochan-b/KeyboardShortcuts)

## Steps

There are three steps to create a keyboard shortcut. The first is to create a `RoutedUICommand` (not sure what it does but some explanation [here](https://joshsmithonwpf.wordpress.com/2008/03/18/understanding-routed-commands/)). The next is to create a `CommandBinding` that defines what to run when the command is called. The final step is to create a `KeyBinding` where we define the keyboard shortcut.

### Step 1

Put the `RoutedUICommand` in the window resources. 

```csharp
    <Window.Resources>
        <RoutedUICommand x:Key="DoSomethingCommand" Text="Do Something" />
    </Window.Resources>
```

### Step 2

Put the `CommandBinding` after the `Window.Resource` section. `RunTest` is a function inside my MainWindow class. 

Be sure to put the `Window.CommandBindings` after `Window.Resources` section in the XAML file. The order matters!

```csharp
    <Window.CommandBindings>
        <CommandBinding Command="{StaticResource DoSomethingCommand}" Executed="RunTest" />
    </Window.CommandBindings>
```

### Step 3

This is where we specify the keyboard shortcut for the command.

```csharp
    <Window.InputBindings>
        <KeyBinding Command="{StaticResource DoSomethingCommand}" Key="L" Modifiers="Ctrl" />
    </Window.InputBindings>
```

## Using the Keyboard Shortcut in a Menu Item

At this point, using `Ctrl+L` anywhere in the app will cause the `RunTest` to be executed. 

I will just show how to use it in a menu item.

```csharp
<MenuItem Header="_Test" Command="{StaticResource DoSomethingCommand}" InputGestureText="Ctrl+L"/>
```

Notice we are using the command instead of click. Click would also work. To get the menu command on the right hand side, we use the `InputGestureText`.

## Conclusion

This is the smallest example with XAML only code to get a keyboard shortcut into a XAML C# program.