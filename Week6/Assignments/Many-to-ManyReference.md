# Many-to-Many Views

**Problem:** MVC doesn't automatically scaffold parts for many-to-many views.

**Solution:** We need to add parts ourselves to deal with them!

_The below suggestions are adapted from [CodeProject](http://www.codeproject.com/Articles/702890/MVC-Entity-Framework-and-Many-to-Many-Relation)_


## ViewModel

First, we are going to create a **ViewModel**. This is a kind of _pseudomodel_ - it doesn't relate directly to a database table, but it is a class we will use to pass data to and from views. We will then have our controller slurp the data out of the ViewModel and translate it into our "traditional" model class.

Let's assume you have an `Outfit` model like the one below:

```cs
public class Outfit
{
    public Outfit()
    {
        this.Accessories = new HashSet<Accessory>();
    }
    public int OutfitId { get; set; }
    public int TopId { get; set; }
    public int BottomId { get; set; }
    public int ShoeId { get; set; }

    public virtual Top Top { get; set; }
    public virtual Bottom Bottom { get; set; }
    public virtual Shoe Shoe { get; set; }

    public virtual ICollection<Accessory> Accessories { get; set; }
}```

First, create a folder called `ViewModels`, and add a class called `OutfitViewModel`.

```cs
public class OutfitViewModel
{
    public Outfit Outfit { get; set; }
    public IEnumerable<SelectListItem> AllAccessories { get; set; }

    private List<int> _selectedAccessories;
    public List<int> SelectedAccessories
    {
        get
        {
            if (_selectedAccessories == null)
            {
                _selectedAccessories = (from a in Outfit.Accessories
                                        select a.AccessoryId).ToList();
            }
            return _selectedAccessories;
        }
        set { _selectedAccessories = value; }
    }
}
```

The `Outfit` property will be used to store the actual `Outfit` object, `AllAccessories` will store a reference to all the possible `Accessory` objects, and the `SelectedAccessories` property will store the `Accessory` objects associated with this particular `Outfit`.

Now we need to update some action methods in the controller to create the `OutfitViewModel` object. In `OutfitsController.cs`, look for the **GET** `Edit` method, and change it as follow:


```cs
// GET: Outfits/Edit/5
public ActionResult Edit(int? id)
{
    if (id == null)
    {
        return new HttpStatusCodeResult(HttpStatusCode.BadRequest);
    }
    Outfit outfit = db.Outfits.Find(id);
    if (outfit == null)
    {
        return HttpNotFound();
    }

    var outfitViewModel = new OutfitViewModel
    {
        Outfit = outfit,
        AllAccessories = from a in db.Accessories
                         select new SelectListItem
                         {
                             Text = a.AccessoryName,
                             Value = a.AccessoryId.ToString()
                         }
    };

    ViewBag.BottomId = new SelectList(db.Bottoms, "BottomId", "BottomName", outfit.BottomId);
    ViewBag.ShoeId = new SelectList(db.Shoes, "ShoeId", "ShoeName", outfit.ShoeId);
    ViewBag.TopId = new SelectList(db.Tops, "TopId", "TopName", outfit.TopId);
    return View(outfitViewModel);
}
```

In the code above, we have created an `OutfitViewModel` object, with the `Outfit` property set to the `Outfit` we looked up in the database, and the `AllAccessories` property set to a LINQ query that creates a bunch of `SelectListItem` objects based upon each of the accessories in the database.

We also need to change the **POST** `Edit` method, as follows:

```cs
// POST: Outfits/Edit/5
// To protect from overposting attacks, please enable the specific properties you want to bind to, for
// more details see http://go.microsoft.com/fwlink/?LinkId=317598.
[HttpPost]
[ValidateAntiForgeryToken]
public ActionResult Edit(OutfitViewModel outfitViewModel)
{
    var outfit = db.Outfits.Find(outfitViewModel.Outfit.OutfitId);
    if (ModelState.IsValid)
    {
        outfit.Accessories.Clear();
        foreach (var accessoryId in outfitViewModel.SelectedAccessories)
        {
            outfit.Accessories.Add(db.Accessories.Find(accessoryId));
        }
        outfit.ShoeId = outfitViewModel.Outfit.ShoeId;
        outfit.TopId = outfitViewModel.Outfit.TopId;
        outfit.BottomId = outfitViewModel.Outfit.BottomId;
        db.SaveChanges();
        return RedirectToAction("Index");
    }
    outfitViewModel = new OutfitViewModel
    {
        Outfit = outfit,
        AllAccessories = from a in db.Accessories
                         select new SelectListItem
                         {
                             Text = a.AccessoryName,
                             Value = a.AccessoryId.ToString()
                         }
    };
    ViewBag.BottomId = new SelectList(db.Bottoms, "BottomId", "BottomName", outfit.BottomId);
    ViewBag.ShoeId = new SelectList(db.Shoes, "ShoeId", "ShoeName", outfit.ShoeId);
    ViewBag.TopId = new SelectList(db.Tops, "TopId", "TopName", outfit.TopId);
    return View(outfitViewModel);
}
```

Now we need to make changes to the view file, `Views/Outfits/Edit.cshtml`:

```html
@model WardrobeJR.ViewModels.OutfitViewModel

@{
    ViewBag.Title = "Edit";
}

<h2>Edit</h2>


@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()

    <div class="form-horizontal">
        <h4>Outfit</h4>
        <hr />
        @Html.ValidationSummary(true, "", new { @class = "text-danger" })
        @Html.HiddenFor(model => model.Outfit.OutfitId)

        <div class="form-group">
            @Html.LabelFor(model => model.Outfit.TopId, "TopId", htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.DropDownListFor(model => model.Outfit.TopId, (SelectList)ViewBag.TopId, htmlAttributes: new { @class = "form-control" })
                @Html.ValidationMessageFor(model => model.Outfit.TopId, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.Outfit.BottomId, "BottomId", htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.DropDownListFor(model => model.Outfit.BottomId, (SelectList)ViewBag.BottomId, htmlAttributes: new { @class = "form-control" })
                @Html.ValidationMessageFor(model => model.Outfit.BottomId, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.Outfit.ShoeId, "ShoeId", htmlAttributes: new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.DropDownListFor(model => model.Outfit.ShoeId, (SelectList)ViewBag.ShoeId, htmlAttributes: new { @class = "form-control" })
                @Html.ValidationMessageFor(model => model.Outfit.ShoeId, "", new { @class = "text-danger" })
            </div>
        </div>

        <div class="form-group">
            @Html.LabelFor(model => model.AllAccessories, "Accessories", new { @class = "control-label col-md-2" })
            <div class="col-md-10">
                @Html.ListBoxFor(model => model.SelectedAccessories, Model.AllAccessories)
            </div>
        </div>

        <div class="form-group">
            <div class="col-md-offset-2 col-md-10">
                <input type="submit" value="Save" class="btn btn-default" />
            </div>
        </div>
    </div>
}

<div>
    @Html.ActionLink("Back to List", "Index")
</div>

@section Scripts {
    @Scripts.Render("~/bundles/jqueryval")
}
```

We are using different ways of accessing the properties, because our actual model is now "hidden" inside our ViewModel. We are using `DropDownListFor` instead of `DropDown`, with some additional parameters. We also need to update the `@model` directive at the top of the file, and add the `ListBoxFor` at the bottom of the form.

We should make similar changes to the `Create` action and view files.

Finally, we need to modify the `Index.cshtml` file in a couple places to show the accessories:

```html
<!-- header row -->
<th>@Html.DisplayNameFor(model => model.Accessories)</th>
```

```html
<!-- inside the foreach loop -->
<td>
    @Html.ListBox("AccessoryId", new SelectList(item.Accessories, "AccessoryId", "AccessoryName"))
</td>
```
