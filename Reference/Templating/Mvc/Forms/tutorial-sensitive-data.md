---
versionFrom: 7.0.0
needsV8Update: "true"
---

# Creating an MVC form using a Partial View

**Applies to: Umbraco 4.10.0+**

_This tutorial will explain how to create a form that contains sensitive data such as email addresses _

## The View Model

The view model that will be used in this tutorial will be as follows:
	
	public class CommentViewModel
	{
	    [Required]
	    public string Name { get; set; }
	
	    [Required]
	    public string Email { get; set; }
	
	    [Required]
	    [Display(Name = "Enter a comment")]
	    public string Comment { get; set; }
	}

This class defines the data that will be submitted and also defines how the data will be validated upon submission. MVC automatically wires up these validation attributes with the front-end so JavaScript validation will automagically occur.

## The Surface Controller

For this tutorial, the Surface controller that we will create will contain one action which is used to accept the POSTed values from the form. In this example, the action will:

*	Check if the model is valid - based on the validation attributes applied to the model above, we will not be performing any custom validation
*	If the model **is not valid**, return the currently rendered Umbraco page (do not redirect). By not redirecting the ViewData is preserved including the ModelState which contains the validation information.
*	If the model **is valid**, add a custom message to the TempData collection and then redirect to the currently rendered Umbraco page. A standard procedure for a web based for is to redirect if the POST is successful. This ensures that the POST cannot be accidentally re-submitted by accidentally pressing F5 (refresh) ... *unfortunately ASP.NET WebForms does not adhere to this rule by default but it 'should' be done in WebForms too.* 

<br/>

	public class BlogPostSurfaceController : Umbraco.Web.Mvc.SurfaceController
	{
		[HttpPost]
		public ActionResult CreateComment(CommentViewModel model)
		{    
		    // model not valid, do not save, but return current Umbraco page
		    if (!ModelState.IsValid)
			{
				// Perhaps you might want to add a custom message to the ViewBag
				// which will be available on the View when it renders (since we're not 
				// redirecting)	    	
		   		return CurrentUmbracoPage();
			}
				    
			// Add a message in TempData which will be available 
			// in the View after the redirect 
			TempData.Add("CustomMessage", "Your form was successfully submitted at " + DateTime.Now)
		
		    // redirect to current page to clear the form
		    return RedirectToCurrentUmbracoPage();		    
		}
	}

## Create a Partial View to render the form

The best way to render a form in MVC is to have a Partial View render the form with a strongly typed model. For this example, we'll create a partial view at location: *~/Views/Partials/BlogCommentForm.cshtml* with a strongly typed model of the model created previously. This example shows how to use the BeginUmbracoForm method with the strongly typed overload to specify which Surface controller and Action to POST to. For brevity, this will auto-scaffold all of the fields for the model using `Html.EditorFor(x => Model)` but you could create the input fields separately if you need more granular control over the markup.

	@model CommentViewModel

	@using(Html.BeginUmbracoForm<BlogPostSurfaceController>("CreateComment"))
	{
		@Html.EditorFor(x => Model)
		<input type="submit"/>
	}

## Render the Partial View

The last step is to render the partial view that we've created in your Umbraco template's view:

	@Html.Partial("BlogCommentForm")

You could also pass in a pre-populated model to pre-populate the fields on the form. For example:

	@Html.Partial("BlogCommentForm", new CommentViewModel() { Name = "Example User" })