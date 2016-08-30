**************************
Creating a Basic Component
**************************

.. UsingIgnition/BasicComponent:

This article will give a practical example of building a content component using Ignition. I’ve called this component “ContentBlurb.” It has a heading, subtitle, and rich text. When added to a page it looks like the following:

.. image:: images\ContentBlurb-View.png
    
Step 1: Create the Sitecore Template
    - Create a new template called “Content Blurb”.
    - Select None for the base template.
    - Go to the content tab of your “Content Blurb” template and select the Ignition fields Heading, Subtitle, and RichContent1 as shown below.

.. image:: images\ContentBlurb-Template.png

Step 2: Create the Component Folder
    - Create a new folder for your component under Ignition/Presentation/Ignition.Sc/Components called “ContentBlurb”.
    - Files related to your component will go in this folder.  This helps minimize scrolling between Controller, Model, and View folders in a traditional MVC solution.
    - The template model will go in the Templates folder for reusability across components.

.. image:: images\ContentBlurb-FolderStructure.png

Step 3: Create the Template Model
    - Create a new template model in the Templates folder called “IContentBlurb”.
    - Make it a public interface that implements the corresponding interfaces for the Ignition fields you added above: IHeading, ISubtitle, and IRichContent1.
    - Set the TemplateId GUID string to match your template item id in Sitecore.
    - Be sure to set AutoMap = true so your model populates correctly. 
    
:: 

    using Glass.Mapper.Sc.Configuration.Attributes;
    using Ignition.Data.Fields;

    namespace Ignition.Sc.Templates
    {
        [SitecoreType(TemplateId = "{44415A4B-4CEC-4194-AA2F-6D8AC0F82B73}", AutoMap = true)]
        public interface IContentBlurb : IHeading, ISubtitle, IRichContent1
        {
        }
    }

Step 4: Create the View Model
    - Create a new view model in your ContentBlurb folder called “ContentBlurbViewModel”.
    - Make it a public class that inherits from “BaseViewModel”.
    - Our view model has one property that uses the template model for our component, IContentBlurb. 
    
::
 
    using Ignition.Core.Mvc;
    using Ignition.Sc.Templates;

    namespace Ignition.Sc.Components.ContentBlurb
    {
        public class ContentBlurbViewModel : BaseViewModel
        {
            public IContentBlurb ContentBlurb { get; set; }
        }
    }

Step 5: Create the Controller
    - Create a new controller in your ContentBlurb folder called “ContentBlurbController”.
    - Make it a public class that inherits from “IgnitionController”.
    - Create an action method to render your component called ContentBlurb as shown below.
    - Make sure you call the generic View<TViewModel>() method with the name of your view model. The Ignition Framework will use this method to autowire your view model with the datasource from your rendering. 
    
::

    using System.Web.Mvc;
    using Ignition.Core.Mvc;
    using Ignition.Core.Repositories;

    namespace Ignition.Sc.Components.ContentBlurb
    {
        public class ContentBlurbController : IgnitionController
        {
            public ActionResult ContentBlurb()
            {
                return View<ContentBlurbViewModel>();
            }
        }
    }

Step 6: Create the View
    - Create a new view in your ContentBlurb folder called “ContentBlurb”.
    - Ignition will find the path to the view file automatically using the name of the controller and the action method at the following path: ~\Components\{Controller}\{Action}.cshtml.
    - Use the @model definition to point to the view model for your component.
    - This example uses GlassMapper to make fields editable in the Experience Editor.
    - Notice the use of a.ContentBlurb.Heading in the call to Glass().Editable. This is accessing the ContentBlurb property of the ContentBlurbViewModel which is an IContentBlurb model, then the Heading property of the IHeading Ignition model inherited in the IContentBlurb model. 
    
:: 

    @model Ignition.Sc.Components.ContentBlurb.ContentBlurbViewModel

    <div class="row contentblurb">
        <div class="col-lg-4">
            <h1>@Html.Glass().Editable(a => a.ContentBlurb.Heading)</h1>
            <h4>@Html.Glass().Editable(a => a.ContentBlurb.Subtitle)</h4>
        </div>
        <div class="col-lg-8">
            @Html.Glass().Editable(a => a.ContentBlurb.RichText1)
        </div>
    </div>

Step 7: Create the Rendering
    - Since this component was created with a controller, create the rendering as a Controller Rendering.
    - Set the following fields ◦Controller – Controller name without “Controller”, i.e., “ContentBlurb”.
    - Action – The action method to call in the controller, i.e., “ContentBlurb”.
    - Datasource Template – The Sitecore template created in step 1.

Step 8: Put it all together
    - Build and deploy your solution.
    - Add an instance of the ContentBlurb component to a page’s presentation details.
    - Publish changes in Sitecore.

The Ignition framework fully embraces the Experience Editor as shown below.

.. image:: images\ContentBlurb-EE.png

And of course the item is easily editable in the Content Editor as well.

.. image:: images\ContentBlurb-CE.png