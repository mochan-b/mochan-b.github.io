---
layout: post
title:  "WPF GridView Cells Formatting with Data Templates and Style Triggers"
date:   2017-06-25 13:00:00 +0000
categories: c# wpf gridview
tags: c# wpf gridview
---

[Link to the repo with the example source code](https://github.com/mochan-b/GridViewCellFormatting)

## Introduction

XAML sometimes feels like a bunch of random magic incantations. There is obviously some detailed model of how things work but sometimes it just gets in the way of doing something simple.

Here, we will explore three simple `GridView` presentations and the data binding for them.

1. Format a row (*here we bold a row whose sum is 21*)
2. Format a column (*background of yellow for the first column*)
3. Format individual cells (*ultra-bold, red cells for value of 6 of the second column*)

![Columns with formatting](/assets/GridViewColumns.png)

## WPF Data Binding Model

We will use very standard and simple code-behind model for `ListView`. We have a `ListView` with a `GridView` with 3 columns called `Data 1`, `Data 2` and `Data 3`. The class that represents a row is then going to be the following:

```csharp
public class MyData
{
    public int data1 { get; set; }
    public int data2 { get; set; }
    public int data3 { get; set; }

    public int sum { get => data1 + data2 + data3; }
}
```

The main class is just adding some data and setting the data context to the MainWindow class itself.

```csharp
public partial class MainWindow : Window
{
    public ObservableCollection<MyData> data { get; set; }

    public MainWindow()
    {
        InitializeComponent();

        this.DataContext = this;

        data = new ObservableCollection<MyData>();
        data.Add(new MyData { data1 = 1, data2 = 2, data3 = 3 });
        data.Add(new MyData { data1 = 5, data2 = 6, data3 = 7 });
        data.Add(new MyData { data1 = 10, data2 = 7, data3 = 4 });
    }
}
```

The XAML is also pretty standard with a `GridView` inside a `ListView`.

```XML
<ListView Grid.Row="0" Grid.Column="0" Margin="10,20,10,10" ItemsSource="{Binding data}" Name="DataList">
```
and each columns is described as follows. 

```xml
<GridViewColumn Width="150" DisplayMemberBinding="{Binding Path=data3}">
    <GridViewColumn.Header>
        <GridViewColumnHeader Tag="Data3">
            <TextBlock>Data 3</TextBlock>
        </GridViewColumnHeader>
    </GridViewColumn.Header>
</GridViewColumn>
```

For the full XAML, please see the GitHub repo.

## Formatting a Row

We want to make the third row (sum of the row is 21) bold. 

The trick is to create a style trigger and bind it to the sum such that when the sum is 21, it triggers the bold style on that row. So, this is essentially a [data trigger](http://wpftutorial.net/Triggers.html). Since the property works on a `ListViewItem` which is a row of data, we put it in the `ListView.Resources`.

```xml
<ListView.Resources>
    <Style TargetType="{x:Type ListViewItem}">
        <Style.Triggers>
            <DataTrigger Binding="{Binding sum}" Value="21">
                <Setter Property="FontWeight" Value="Bold" />
            </DataTrigger>
        </Style.Triggers>
    </Style>
</ListView.Resources>
```

## Formatting a Column

We want to make the background of the first column yellow.

The trick is to use a [`Data Template`](http://wpftutorial.net/DataTemplates.html). The idea of a data template is that you can specify exactly what is displayed in a cell as a set of XAML controls. The default is just a plain old `TextBlock`. We are just going to replace it with a `TextBlock` with a yellow background.

```xml
<GridViewColumn Width="150">
    <GridViewColumn.CellTemplate>
        <DataTemplate>
            <TextBlock x:Name="Txt1" Text="{Binding data1}" Background="Yellow" />
        </DataTemplate>
    </GridViewColumn.CellTemplate>
    <GridViewColumn.Header>
        <GridViewColumnHeader Tag="Data1">
            <TextBlock>Data 1</TextBlock>
        </GridViewColumnHeader>
    </GridViewColumn.Header>
</GridViewColumn>
```

## Formatting a Cell

Formatting a cell ends up using a style trigger with data trigger and a data template together. You just put the style trigger inside of the `DataTemplate` `TextBlock`.

```xml
<GridViewColumn.CellTemplate>
    <DataTemplate>
        <TextBlock x:Name="Txt2" Text="{Binding data2}" >
            <TextBlock.Style>
                <Style TargetType="TextBlock">
                    <Style.Triggers>
                        <DataTrigger Binding="{Binding data2}" Value="6">
                            <Setter Property="Foreground" Value="Red"/>
                            <Setter Property="FontWeight" Value="UltraBold"/>
                        </DataTrigger>
                    </Style.Triggers>
                </Style>
            </TextBlock.Style>
        </TextBlock>
    </DataTemplate>
</GridViewColumn.CellTemplate>
```

## Conclusion

`Style Triggers` and `Data Templates` are a bigger concept in XAML and extend beyond just `GridView`. But, if you're working with `GridView` and just need to get a few things working with it, the above are the semi-magic incantations.