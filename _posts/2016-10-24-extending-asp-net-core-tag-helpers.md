---
layout: post
title: Creating a readonly textarea by extending Asp.Net Core input tag helper
date: 2016-10-24 00:11:29.000000000 -05:00 
tags: [tag helpers, AspNetCore]
---

Continuing the "mostly code" series...

Occasionally, you might want to render a disabled form element to the user in your views. Generally, to render a disabled element,
 you need to add the disabled attribute to the element.Interestingly all the below code will render a disabled textarea.
{% highlight c# %}
<textarea disabled="disabled"></textarea>
<textarea disabled="true"></textarea>
<textarea disabled="false"></textarea>
<textarea disabled="iReallDontWantThisToHappen"></textarea>
{% endhighlight %}
The existence of disabled attribute will make the form element disabled irrespective of what value you provide.

With ASP.NET core, you have the input tag helpers to render the form elements. to render a text area, you will do like this.
{% highlight c# %}
<textarea asp-for="CommentBody"></textarea>
{% endhighlight %}
where `CommentBody` is a property of your view model to which your view is strongly typed to.

Disabled textarea
====

The input tag helper does not have direct support for disabled attribute. But we can create a custom input helper which does that. We do not need to create all the functionalities for the 
existing TextArea helper. We may simply inherit from that and add this readonly behavior.

Simply create a new class inheriting from TextAreaTagHelper. We will add a new attribute called "is-diabled" which is of boolean type. Based on the value of this, we will conditionally add the dissabled attribute the textarea.

    [HtmlTargetElement("textarea", Attributes = ForAttributeName)]
    public class MyCustomTextArea : TextAreaTagHelper
    {
        private const string ForAttributeName = "asp-for";

        [HtmlAttributeName("asp-is-disabled")]
        public bool IsDisabled { set; get; }

        public MyCustomTextArea(IHtmlGenerator generator) : base(generator)
        {
        }

        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            if (IsDisabled)
            {
                var d = new TagHelperAttribute("disabled", "disabled");
                output.Attributes.Add(d);
            }
            base.Process(context, output);
        }
    }

Now to use this custom textarea helper, you need to call the `addTagHelper` method in `_ViewImports.cshtml`.


	@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
	@addTagHelper *, MyAppAssemblyName

Replace `MyAppAssemblyName` with your assembly name.

Now in your views, you can use this tag helper like this

	<textarea asp-for="PostBody" asp-is-disabled="true" disabled="disabled"></textarea>
Or if your view model has a boolean property called IsReadOnly
    <textarea asp-for="PostBody" asp-is-disabled="@Model.IsReadOnly"></textarea>


Cheers !