*********************
Adding Business Logic
*********************

.. UsingIgnition/BusinessLogic:

In this article I will continue using my “ContentBlurb” component I created in “:doc:`BasicComponent`“.  I will add some business logic to our component using an agent.  I want to display a default subtitle if that field is empty.  We could put our business logic in the controller, but that makes the controller bloated and doing more work than it should following good separation of concern principles.   Recall that our Content Blurb component currently looks like the following on the screen.

.. image:: images\ES-DFI-BL-ContentBlurb.png

All of the Ignition fields inherit from “IModelBase” by default.  IModelBase gives us access to the Sitecore fields: GUID, Language, Uri, DisplayName, Version, Path, Name, and Url of our datasource item.  When your template model inherits from at least one Ignition field, you do not need to specifically inherit from IModelBase.  In order to access the other Sitecore fields: Sortorder, Created, Updated, FullPath, FullUrl, TemplateId, and TemplateName, we need our template model to also inherit from “IModelBaseWithMetadata”.

Step 1: Update the Template Model
    - Update your template model “IContentBlurb”.
    - Add “IModelBaseWithMetadata” to the list of inheritance.
    - Note the additional “using Ignition.Core.Models.BaseModels” statement needed to use the “IModelBaseWithMetadata” interface.

:: 

    using Glass.Mapper.Sc.Configuration.Attributes;
    using Ignition.Core.Models.BaseModels;
    using Ignition.Data.Fields;

    namespace Ignition.Sc.Templates
    {
        [SitecoreType(TemplateId = "{44415A4B-4CEC-4194-AA2F-6D8AC0F82B73}", AutoMap = true)]
        public interface IContentBlurb : IHeading, ISubtitle, IRichContent1, IModelBaseWithMetadata
        {

        }
    }

Step 2: Create the Agent
    - Create a new agent in your ContentBlurb folder called “ContentBlurbAgent”.
    - Make it a public class that inherits from “Agent<ContentBlurbViewModel>”, from the generic “Agent<TViewModel>”.
    - Override the PopulateModel method and add your business logic.  This function is called automatically by the Ignition framework after it binds the datasource to your view model.  This gives you a chance to check or update any of the properties of the view model before it gets sent to the view to be rendered.
    - Here, I am checking to see if the subtitle field is null or empty.  If so, I am assigning the “Updated” value from the datasource.
    - Notice that I’m casting the datasource as “IModelBaseWithMetadata”.  If you don’t do the cast, the compiler assumes the datasource is an “IModelBase”, and won’t find the “Updated” property.

.. image:: images\ES-DFI-BL-FolderStructure.png
 
::

    using Ignition.Core.Models.BaseModels;
    using Ignition.Core.Mvc;
    using Sitecore.Mvc.Extensions;

    namespace Ignition.Sc.Components.ContentBlurb
    {
        public class ContentBlurbAgent : Agent<ContentBlurbViewModel>
        {
            public override void PopulateModel()
            {
                if (ViewModel.ContentBlurb.Subtitle.Trim().IsEmptyOrNull())
                {
                    ViewModel.ContentBlurb.Subtitle = (Datasource as IModelBaseWithMetadata).Updated.ToLongDateString();
                }
            }
        }
    }

Step 3: Update the Controller
    - Update your controller “ContentBlurbController”.
    - We change our call from View<TViewModel> to View<TAgent, TViewModel>() passing the name of our agent along with our view model.  When you only pass the view model, Ignition uses an internal “SimpleViewAgent” in the background that has no business logic.
    - It is easy to miss this step and wonder why your custom business logic is not being called when you are debugging!

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
                return View<ContentBlurbAgent, ContentBlurbViewModel>();
            }
        }
    }

Step 4: Putting it all together
    - Build and deploy your solution.
    - Edit the content of your item and leave the subtitle field blank.
    - Publish changes in Sitecore.

.. image:: images\ES-DFI-BL-ContentEditor.png

The subtitle displays the date the content item was last updated.

.. image:: images\ES-DFI-BL-DefaultSubtitle.png
