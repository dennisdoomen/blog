---

title: 9 simple practices for writing better object-oriented code
date: '2015-10-25T16:47:00.003+01:00'

tags: 
 - maintainability
 - coding conventions
modified_time: '2015-10-25T16:48:34.014+01:00'
---

Consider a fantasy game that must track a collection of items, each having a certain amount of *quality* (or value) that increases or decreases after time passes. This collection contains the following six items:

*    +5 Dexterity Vest
*   Aged Brie
*   Elixir of the Mongoose,
*   Sulfuras, Hand of Ragnaros
*   Backstage passes to a TAFKAL80ETC concert
*   Conjured Mana Cake

How much each item's quality increases or decreases over time depends on an intricate algorithm that involves the *quality* and *sell-in* (days) properties of each item. Fortunately a prior C# developer encoded this for us. Let's see what this algorithm looks like:

```csharp 
private static void UpdateQuality(Item item)
{
    if (item.Name != "Aged Brie" && item.Name != "Backstage passes to a TAFKAL80ETC concert")
    {
        if (item.Quality > 0)
        {
            if (item.Name != "Sulfuras, Hand of Ragnaros")
            {
                item.Quality = item.Quality - 1;
            }
        }
    }
    else
    {
        if (item.Quality < 50)
        {
            item.Quality = item.Quality + 1;

            if (item.Name.Equals("Backstage passes to a TAFKAL80ETC concert"))
            {
                if (item.SellIn < 11)
                {
                    if (item.Quality < 50)
                    {
                        item.Quality = item.Quality + 1;
                    }
                }

                if (item.SellIn < 6)
                {
                    if (item.Quality < 50)
                    {
                        item.Quality = item.Quality + 1;
                    }
                }
            }
        }
    }

    if (!item.Name.Equals("Sulfuras, Hand of Ragnaros"))
    {
        item.SellIn = item.SellIn - 1;
    }

    if (item.SellIn < 0)
    {
        if (item.Name != "Aged Brie")
        {
            if (item.Name != "Backstage passes to a TAFKAL80ETC concert")
            {
                if (item.Quality > 0)
                {
                    if (item.Name != "Sulfuras, Hand of Ragnaros")
                    {
                        item.Quality = item.Quality - 1;
                    }
                }
            }
            else
            {
                item.Quality = item.Quality - item.Quality;
            }
        }
        else
        {
            if (item.Quality < 50)
            {
                item.Quality = item.Quality + 1;
            }
        }
    }
}
```  

So? What do you think? If you want, you can find the full sources of this version [here](https://github.com/dennisdoomen/objectcalisthenics/tree/master). Is it readable enough for you? I hope not, because that would render this entire article useless. So let's use this opportunity to introduce the first practice: **use only one level of indention per method**. In other words, a method can have an `if` statement as long as it is not nested in another construct, other than the class member itself. Obviously the code snippet above violates this practice big time, so let's rewrite the algorithm.

```csharp
private static void UpdateQuality(Item item)
{
    if ((item.Name != "Aged Brie") && (item.Name != "Backstage passes to a TAFKAL80ETC concert"))
    {
        ReduceQualityUnlessNameIsSulfuras(item);
    }

    else
    {
        HandleInsufficientQuality(item);
    }

    if (!item.Name.Equals("Sulfuras, Hand of Ragnaros"))
    {
        item.SellIn = item.SellIn - 1;
    }

    if (item.SellIn < 0)
    {
        HandleInsufficientSellIn(item);
    }
}

private static void ReduceQualityUnlessNameIsSulfuras(Item item)
{
    if ((item.Quality > 0) && (item.Name != "Sulfuras, Hand of Ragnaros"))
    {
        item.Quality = item.Quality - 1;
    }
}

private static void HandleInsufficientQuality(Item item)
{
    if (item.Quality < 50)
    {
        item.Quality = item.Quality + 1;

        HandleBackstagePasses(item);
    }
}

private static void HandleBackstagePasses(Item item)
{
    if (item.Name.Equals("Backstage passes to a TAFKAL80ETC concert"))
    {
        UpdateQualityBasedOnQualityAndSellIn(item);
    }
}

private static void UpdateQualityBasedOnQualityAndSellIn(Item item)
{
    if ((item.SellIn < 11) && (item.Quality < 50))
    {
        item.Quality = item.Quality + 1;
    }

    if ((item.SellIn < 6) && (item.Quality < 50))
    {
        item.Quality = item.Quality + 1;
    }
}

private static void HandleInsufficientSellIn(Item item)
{
    if (item.Name != "Aged Brie")
    {
        HandleItemsThatAreNotAgedBrie(item);
    }
    else
    {
        HandleOtherItems(item);
    }
}

private static void HandleItemsThatAreNotAgedBrie(Item item)
{
    if (item.Name != "Backstage passes to a TAFKAL80ETC concert")
    {
        IfNotSulfurasAndQualityIsSufficient(item);
    }
    else
    {
        item.Quality = item.Quality - item.Quality;
    }
}

private static void IfNotSulfurasAndQualityIsSufficient(Item item)
{
    if ((item.Quality > 0) && (item.Name != "Sulfuras, Hand of Ragnaros"))
    {
        item.Quality = item.Quality - 1;
    }
}

private static void HandleOtherItems(Item item)
{
    if (item.Quality < 50)

    {
        item.Quality = item.Quality + 1;
    }
}
```
  
I tried to replace all the nested constructs with a simple [Extract Method](http://refactoring.com/catalog/extractMethod.html) refactoring pattern. Did it help? I suspect you've discovered by now that backstage passes increase in value the closer you get to their end-of-life. You may also have noticed that Sulfarus never looses its value and that the value of an item never exceeds 50. But we can do better. Let's introduce the practices **Don't Use The Else Keyword and Use Only Dot Per Line**.
 

Most of the dots in the code snippet above are caused by the reference to the `item` variable, which is a sign of [Feature Envy](https://sourcemaking.com/refactoring/smells/feature-envy) code. Similarly, you could say most of the `else`s are caused by the procedural nature of this code example. And what else could be the the best solution to this than introducing a class per item? This is the revised implementation for the Elixer of the Mongoose item. 

```csharp
public class ElixirOfTheMongoose : Item
{
    public ElixirOfTheMongoose(int sellIn, int quality)
        : base("Elixir of the Mongoose", sellIn, quality)
    {
    }

    public override void UpdateQuality()
    {
        if (Quality > 0)
        {
            Quality = Quality - 1;
        }

        SellIn = SellIn - 1;

        if (SellIn < 0 && Quality > 0)

        {
            Quality = Quality - 1;
        }
    }
}
```  

And now the algorithm is clear; each day the quality decreases gradually, but until the end-of-life has been reached, after which the quality decreases twice as fast. You could say that these little practices force you to consider a more object-oriented design, don't you agree? But, if you take a closer look, you'll notice something else. Apparently, the quality can never become negative.

You could have solved this by using an `unsigned int`, but by applying the **Wrap All Primitives and Strings** practice you can do better. By wrapping the two numeric properties of the `Item` base-class in classes with value-type semantics such as overloads for `Equals`, `GetHashCode`, various equality operators and by adding intention revealing members like `Decrease`, `IsOverdue` and `Decrement`, you can rewrite the `UpdateQuality` method as:

```csharp
public override void UpdateQuality()
{
    Quality = Quality.Decrease();

    SellInDays = SellInDays.Decrement();

    if (SellInDays.IsOverdue)

    {
        Quality = Quality.Decrease();
    }
}
```
  
We're left with four lines of code. Few enough to almost instantaneously understand the algorithm. But we're not done yet. Suppose we want to extract some characteristics from a collection of items such as the highest value item as well as the items that are overdue. We could take that `List` from the `GildedRose` class and do the following.
  
```csharp
Item highestValueItem = items.OrderBy(i => i.Quality).Last();

IEnumerableitemsOverdue = items.Where(i => i.SellInDays.IsOverdue);
```  

But then we're not only violating **Use Only Dot Per Line again**. The mere fact that we use a simple List is violation of another practice, which is the one named **First Class Collections**. And now that I think of it, how would one call a collection of items that need to be sold? Ah, what about an *inventory*. So let's create the `Inventory` class.

```csharp
public class Inventory : IEnumerable
{
    private readonly HashSet items = new HashSet();

    public Item HighestValued
    {
        get { return items.OrderBy(i => i.Quality).Last(); }
    }

    public IEnumerable Overdue
    {
        get { return items.Where(i => i.SellInDays.IsOverdue).ToArray(); }
    }

    public IEnumerator GetEnumerator()
    {
        return items.GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }

    public void Add(Item item)
    {
        items.Add(item);
    }

    public void HandleDayChanges(int nrOfDays)
    {
        for (int i = 0; i < nrOfDays; i++)

        {
            foreach (var item in items)

            {
                item.HandleDayChange();
            }
        }
    }
}
```  

Which means we can rewrite the original two calls like this:

  
```csharp
Item highestValueItem = inventory.HighestValued;

IEnumerableitemsOverdue = inventory.OverDue;
```

Wrapping simple collections in first-class types is actually a very smart thing to do. How many times have your classes exploded in size with private methods which only task was to modify or query your private collection field? If you would have applied the practices I mentioned up to now, I'm pretty sure you agree you would have ended up with better distribution of responsibilities. It might feel all like overkill, but since I've learned about these practices, it has really helped me create better object-oriented code.

Another practice, aptly called **Don't Abbreviate**, tells us not to abbreviate names of classes, members and variables. This practice can be confusing, because the question you should ask yourself should be why you want to abbreviate names in the first place. I guess it's either because you couldn't find a shorter name to describe your class, or because you are trying to avoid repeating the same words over and over. The former usually happens to me if I'm trying to name a class that has too many responsibilities (hence the  Single Responsibility Principle in [SOLID](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)). The latter could be, and usually is, caused by not following **First Class Collections** and **Wrap All Primitives and Strings**.

It's awesome to realize how these practices work together so well to accomplish another practice named **Keep All Entities Small**. Of course, *small* is not an objective measurement, but I highly doubt anyone would think the code example I started this post with is small. And that's the purpose of this practice. Whenever your class turns into anything but small, start applying the previous practices to make it small again. One heuristic that can help you detect this as early as possible is to practice **No Classes With More Than Two Instance Variables**. I know that is going to be difficult to meet. Hey, you might even call it unrealistic and overkill. Regardless, I challenge you to consider the other practices whenever you pass the first two instance fields.

Now consider the following implementation of the Item class.

```csharp
public abstract class Item
{
    private readonly string name;

    protected Item(string name, SellInDays sellIn, Quality quality)
    {
        this.name = name;

        Quality = quality;

        SellInDays = sellIn;
    }

    public SellInDays SellInDays { get; set; }

    public Quality Quality { get; set; }

    public string Name
    {
        get { return name; }
    }

    public int DaysOverdue
    {
        get { return SellInDays.DaysOverdue; }
    }

    public abstract void HandleDayChange();

    public override string ToString()
    {
        return name;
    }
}
```  

Technically, this class doesn't have any fields (instance variables in C#), but the properties used here are just compiler-generated getters and setters around a hidden field. So in reality this class violates the two instance practice with one field. But do we need this getters and setters at all? If you look closely at the code, you'll see that those properties are either used for sorting or filtering as illustrated in the previous practice, or to compare them in unit tests. Quite often, properties open up the possibility for calling code to apply algorithms that really belong inside the class that exposes the properties. So you could say that they are generally not a favorable solution, hence the **No Getters/Setters/Properties** practice.

In our case, this theory is actually true. For instance, two items are semantically equivalent if their `Quality`, `SellinDays` and `Name` are the same, which is what the unit tests rely on. We can easily replace that behavior by the correct implementation of `Equals`. Likewise, sorting items based on the quality of these properties can be replaced by exposing an implementation of `IComparer`. The only one that we can't really remove is the `DaysOverdue` property. And I don't mind too much either. Being able to ask an item how many days it's due sounds like something you should be able to ask an item in this domain. Anyway, this the current implementation, although I'm discovering refactoring opportunities every time I look at the code.

```csharp
public abstract class Item
{
    private readonly string name;

    private Days shelfLife;

    private Quality quality;

    protected Item(string name, Days shelfLife, Quality quality)

    {
        this.name = name;
        this.quality = quality;
        this.shelfLife = shelfLife;
    }

    public bool IsExpired
    {
        get { return shelfLife < new Days(0); }
    }

    public Days DaysOverdue
    {
        get { return (shelfLife > new Days(0)) ? new Days(0) : -shelfLife; }
    }

    public static IComparer ByQualityComparer
    {
        get { return new QualityComparer(); }
    }

    public abstract void OnDayHasPassed();

    public override string ToString()
    {
        return string.Format("{0} (quality {1}, sell in {2} days)", name, quality, shelfLife);
    }

    protected bool Equals(Item other)
    {
        return shelfLife.Equals(other.shelfLife) && string.Equals(name, other.name) && quality.Equals(other.quality);
    }

    public override bool Equals(object obj)
    {
        if (ReferenceEquals(null, obj))

        {
            return false;
        }

        if (ReferenceEquals(this, obj))

        {
            return true;
        }

        return Equals((Item)obj);
    }

    public override int GetHashCode()
    {
        unchecked

        {
            int hashCode = shelfLife.GetHashCode();

            hashCode = (hashCode \*397) ^ name.GetHashCode();

            hashCode = (hashCode \*397) ^ quality.GetHashCode();

            return hashCode;
        }
    }

    protected void IncreaseQuality()
    {
        quality = quality.Increase();
    }

    protected void DecreaseQuality()
    {
        quality = quality.Decrease();
    }

    protected void Devaluate()
    {
        quality = new Quality(0);
    }

    protected void ReduceShelfLife()
    {
        shelfLife = shelfLife.ReduceByOneDay();
    }

    protected bool IsDueWithin(Days days)
    {
        return shelfLife < days;
    }

    private class QualityComparer : IComparer
    {
        public int Compare(Item x, Item y)

        {
            return x.quality.CompareTo(y.quality);
        }
    }
```

So what do you think? Do these practices, collectively called [Object Calisthenics](http://www.slideshare.net/dennisdoomen/object-calisthenetics), make sense? Did I motivate you to try them right away? If so, tell me your success stories by participating in the Disqussions below. And please follow me at [@ddoomen](https://www.twitter.com/ddoomen) to get regular updates on my everlasting quest for better solutions.
