---
layout: post
title: Asp.Net Core Custom tag helper for gravatar image from Email
date: 2016-10-01 00:11:29.000000000 -05:00 
tags: [tag helpers, aspnetcore, gravatar]
---





As we have [excellent documentation](https://docs.asp.net/en/latest/mvc/views/tag-helpers/intro.html) on the official documentation page explaining  in detail what tag helpers are and what problem they solve, i am not going to repeat that here.This is going to be mostly code :)

While rewriting TeamBins, [the open source issue tracking software](https://github.com/kshyju/ProjectPlanningTool) from ASP.NET MVC5 to Asp.Net core, I decided to convert an exisitng gravatar html helper to a tag helper. Here it goes.

{% highlight c# %}
[HtmlTargetElement("span", Attributes = EmailAttributeName)]
public class GravatarImageTagHelper : TagHelper
{
    private const string SizeAttributeName = "image-size";
    private const string EmailAttributeName = "gravatar-email";
    private const string AltTextAttributeName = "alt";

    const string GravatarBaseUrl = "http://www.gravatar.com/avatar.php?";

    [HtmlAttributeName(EmailAttributeName)]
    public string Email { get; set; }

    [HtmlAttributeName(AltTextAttributeName)]
    public string AltText { set; get; }

    [HtmlAttributeName(SizeAttributeName)]
    public int? Size { get; set; }

    private string ToGravatarHash(string email)
    {
        var encoder = new UTF8Encoding();
        var md5 = MD5.Create();
        var hashedBytes = md5.ComputeHash(encoder.GetBytes(email.ToLower()));
        var sb = new StringBuilder(hashedBytes.Length * 2);

        for (var i = 0; i < hashedBytes.Length; i++)
            sb.Append(hashedBytes[i].ToString("X2"));

        return sb.ToString().ToLower();
    }

    private string ToGravatarUrl(string email, int? size)
    {

        if (string.IsNullOrEmpty(email) || string.IsNullOrEmpty(email.Trim()))
            throw new ArgumentException("The email is empty.", nameof(email));

        var sb = ToGravatarHash(email);

        var imageUrl = GravatarBaseUrl + "gravatar_id=" + sb;
        if (size.HasValue)
            imageUrl += "?s=" + size.Value;

        return imageUrl;

    }


    public override void Process(TagHelperContext context, 
                                                        TagHelperOutput output)
    {
        var str = new StringBuilder();
        var url = ToGravatarUrl(this.Email, this.Size);
        str.AppendFormat("<img src='{0}' alt='{1}' />", url, AltText);
        output.Content.AppendHtml(str.ToString());

    }
 }
{% endhighlight %}


This tag helper can be invoked on any span elements. Email attribute name (`gravatar-email`) is the only required field. This can be called in another view like this

    <span gravatar-email="YourValidEmailAddressHere" image-size="26"></span>
    
When razor executes this view code, it will render the below html markup

    <span><img src="http://www.gravatar.com/avatar.php?gravatar_id=YourEmailHash?s=26" alt=""></span>
    
`YourEmailHash` will be replaced with the hash generated from the email address.

If email is a property of your view model you will use that property as the value of `gravatar-email` attribute.

    <span gravatar-email="@Model.EmailAddress" image-size="26"></span>
   
You also need to add an `addTagHelper` directive call in your `_ViewImports.cshtml` file for the tag helper to work.  The second argument is the assembly name where your tag helper is defined. In my case, It is in my `TeamBinsCore.Web` project (and assembly)

    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
    @addTagHelper *, TeamBinsCore.Web
    
Cheers ! 