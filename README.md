Onenote Object Model (onom)
====

The OneNote Object Model exposed via C#. This project contains the following sub directories:

* OneNoteObjectModel - An assembly exposing the ObjectModel
* ListNotebooks - An example showing listing the notebooks. 

Usage snippets:
==========

List Notebooks
----------------
Many OneNote APIs, like GetHierarchy, are not conveniently typed, so ONOM implements typed helpers. which you can use to list notebooks in a type safe manner:

```csharp
// Access your notebooks and sections using Linq

var ona = new OneNoteObjectModel.OneNoteApp();
var notebook = ona.GetNotebooks().Notebook.First(n=>n.name == "BlogContentAndResearch");
var section = ona.GetSections(notebook).First(s=>s.name == "Current");
```

Erase Blank Sections and Pages
---------------------
It's easy to accidently create new pages and sections, this snippet finds and erases them using the IsDefaultUnModified extension methods: 

```csharp
// Find and erase blank pages
var ona = new OneNoteObjectModel.OneNoteApp();
var blankSections = ona.GetNotebooks().Notebook.SelectMany(n=>n.PopulatedSections(ona)).Where(s=>s.IsDefaultUnmodified(ona));
blankSections.ToList().ForEach(s=>ona.OneNoteApplication.DeleteHierarchy(s.ID));

// You can also find and erase blank pages, but that's slow: 
var blankPages = ona.GetNotebooks().Notebook.SelectMany(n=>n.PopulatedSections(ona).SelectMany(s=>s.Page)).Where(s=>s.IsDefaultUnmodified(ona));
blankPages.ToList().ForEach(p=>ona.OneNoteApplication.DeleteHierarchy(p.ID));
```


Create a new "Daily Page" from a template:
----------------
It's great to create a new page per day using a template. But updating page XML is painful, this snippet create a new page by using the ClonePage method:

```csharp

// Clone the daily template into a page named with today's date.
var ona = new OneNoteObjectModel.OneNoteApp();

var templateForDay = ona.GetNotebooks().Notebook.First(n=>n.name == "Templates")
					  .PopulatedSections(ona).First(s=>s.name == "Default")
					  .Page.First(p=>p.name == "Daily");
					  
var sectionForDayPage = ona.GetNotebooks().Notebook.First(n=>n.name == "BlogContentAndResearch")
					 .PopulatedSections(ona).First(s=>s.name == "Current");		

var newPageTitle = DateTime.Now.Date.ToShortDateString();
if (sectionForDayPage.Page.Any(p=>p.name == newPageTitle))
{
	Console.WriteLine("Today's template ({0}) has already been created.",newPageTitle);
}
else
{
	ona.ClonePage(sectionForDayPage,templateForDay,newPageTitle);
	Console.WriteLine("Created today's template page ({0}).",newPageTitle);
}

```


TODO List:

* Create a NuGet project for easy access from other projects and LinqPad

