---
title: 'On Tableviews'
layout: post
date: 2017-05-04
tags: swift
---

You’re always trying to show tables of something to people. Always.

The *UITableView & UITableViewController* classes, help you specifically with
that.

In this blog, we’ll discover how to use them.

#### **UITableView**

The UITableView class of the iOS API, is quite robust and powers millions of
tables on iOS across the world. 

*UITableView* is a view component that can be used to display a list of items in
a table, and can be embedded into a view. If you’re simply looking to implement
a quick hassle-free and a straight forward table, the *UITableViewController*
object is the way to go.

#### **UITableViewController**

*UITableViewController = UITableView + UIViewController*

The UITableViewController is a subclass of UIViewController that manages a table
view, and can be used independantly.

#### **How to?**

Now that we have an idea about how tables are used, let’s look at how to
implement them.

The *UITableView* requires two things to function:

* Datasource
* Delegate

The Datasource provides all things content, or the data for the table, and other
information that the table needs to construct the table.

The Delegate manages the table configuration, selection, editing, and other
interactions.

Usually the view controller that manages the tableview acts as the Datasource
and Delegate for the corresponding table. This is accompolished by making the
View Controller adopt the following protocols:

* **UITableViewDataSource**: *for the datasource methods*

* **UITableViewDelegate**: *for the delegate methods*


I have personally only used IB and AutoLayout to create my UIs so far, but to
move away from this practice and to start writing UIs programmatically, I’m
going to stick to creating UIs programmatically in these posts that I write in
the hope that I learn along the way. If you need help setting up XCode without
storyboards, give
[this](https://medium.com/ios-os-x-development/ios-start-an-app-without-storyboard-5f57e3251a25)
article a quick read.


#### **Step I:**

**Prepare Items & Table view**

In *ViewController.swift*, intialize two variables — one that holds a new
*UITableView* object and the other that holds the items that you’re going to
display in the table.

{% highlight swift %}
let tableView: UITableView = UITableView()
let items: [String] = ["One", "Two", "Three"]
{% endhighlight %}

#### **Step II:**

**Configure TableView**

Before adding our tableview to our view, there are three things that we need to
take care of:

* The Table View’s **Frame** in the view
* The Table View’s **Datasource** and **Delegate**
* The Table View’s **Cell Reuse Identifier**

In your *viewWillAppear()* method:

1.  **Setup Frame:**

{% highlight swift %}
let screenSize: CGRect = UIScreen.main.bounds
let width = screenSize.width
let height = screenSize.height

tableView.frame = CGRect(x: 0, y: 0, width: width, height: height)
{% endhighlight %}

There are two things happening here:

* We’re obtaining the screen’s width and height using the bounds property of our
UIScreen — **UIScreen.main.bounds**
* Setting our Table View’s frame using the *frame* property on our table view —
**tableview.frame**

2. **Set Table View’s Datasource and Delegate**

After setting the tableview’s frame, we may now set the tableview’s datasource
and delegate to our own class that implements the tableview. In this case, *self
*points to our *ViewController *class.

{% highlight swift %}
tableView.dataSource = self
tableView.delegate = self
{% endhighlight %}

After setting the tableview’s datasource and delegate, its important we
implement the datasource and delegate methods.

3. **Set Reuse Identifier**

{% highlight swift %}
tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
{% endhighlight %}

Set the reuse identifier that you would like to use for the cell, so the cell
instance can be reused during run time.

#### Step III:

**Implement Datasource & Delegate Methods**

{% highlight swift %}
  extension ViewController: UITableViewDataSource {
    func numberOfSections(in tableView: UITableView) -> Int {
      return 1 
    }

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
      return items.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
      let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
      cell.textLabel?.text = items[indexPath.row]
      return cell
    }
  }
{% endhighlight %}

What’s happening here?

I like to use extensions to keep things clean and modular —

We first create an extension to our class, and make the class adopt to the
*UITableViewDataSource* protocol.

The tableview requests its datasource for information regarding the structure of
the table to build it.

Within the extension, there are three methods that have been implemented:

{% highlight swift %}
numberOfSections(in tableView: UITableView) -> Int
{% endhighlight %}

This specifies the number of sections that the table is going to have.

{% highlight swift %}
tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int
{% endhighlight %}

This returns the total number of rows the table will have. Ideally we return the
count of our datasource (in our case an array of items) — *items.count*

{% highlight swift %}
tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell
{% endhighlight %}

This returns an instance of the tableview cell for an index path. This is where
the data binding usually happens.

{% highlight swift %}
let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
cell.textLabel?.text = items[indexPath.row]
return cell
{% endhighlight %}

* A tableview cell is dequeued for the correspoding indexpath
* The cell’s textLabel property holds the text from our array
* The Cell is returned

#### Step IV:

**Adding Table View to the View**

Now that our Table View has been setup, all that’s left is adding it to our
view.

In your *viewDidLoad() *method:

{% highlight swift %}
view.addSubview(tableView)
{% endhighlight %}

Now when you Build and Run, you should see your table populated with the data
from the items array.

That’s how a simple table is created, a complex one builds over the same
concepts and a cell could be designed to hold text, images, and even videos.

I hope that was useful.


