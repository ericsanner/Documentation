*********************
Adding View Parameter
*********************

.. UsingIgnition/ViewParameter:

In this article I will continue using my “ContentBlurb” component I created in “:doc:`BasicComponent`” and updated in “:doc:`BusinessLogic`“.  I will add a view parameter that can position the heading on the left or the right of the rich text field.  View parameters give content authors some limited design options that can be changed for each individual instance of a component, regardless of the datasource.  View parameters should only affect the view.  You should never use view parameters to store data as they can not be versioned or localized.  The view parameter is created by using a rendering parameters template and assigning the template in the template for our component.  Recall that our Content Blurb component currently looks like the following on the screen.

.. image:: images\ViewParameter-ViewLeft.png

Step 1: Create the Parameter Option Values
    - Create a new folder outside the main site content root called “Content Blurb” using the “SettingsFolder” Ignition template.  I’ve put it under /SiteSettings/ComponentStyles to keep all of my rendering parameter options for all of my components grouped together in one place.
    - Create a new folder under “Content Blurb” called “Position” using the “SettingsFolder” Ignition template.  Putting the options related to one view parameter in a folder makes it easier to add other view parameters to this component in the future.
    - Create your option values using the “StringSetting” Ignition template.  The name of the item is what users will see in a drop down.  The string entered into the input box “StringSetting” is value Sitecore will make available in your code.

.. image:: images\ViewParameter-Option.png

Step 2: Create the Rendering Parameters Template
    - Create a new rendering parameters template called “Heading Options”.
    - Select None for the base template.
    - Go to the content tab of your “Heading Options” template and select the Ignition item “RenderingParamBase” from the Ignition folder.
    - Go to the builder tab of your “Heading Options” template.  Name the section “CSS Settings” and add a field called “HeadingPosition”.
    - Select “Droplink” for the type.  A droplink renders to the user as an html dropdown select box populated with items in the SettingsFolder pointed to by the source GUID.

        - Make sure not to select “droplist”!  A droplist stores an item’s name as a value, we want to reference the selected item using its GUID.

.. image:: images\ViewParameter-Template1.png
    
.. image:: images\ViewParameter-Template2.png

Step 3: Update the Rendering
    - Update your “ContentBlurb” controller rendering.
    - Select “Heading Options” for the “Parameters Template” field.

.. image:: images\ViewParameter-Rendering.png

Step 4: Create the Parameters Model
    - Create a new parameters model in your ContentBlurb folder called “IContentBlurbParams”.
    - Make it a public interface that inherits from “IParamsBase”.
    - Add the “HeadingPosition” property. 

        - Notice this is an “IStringSetting” to match the Ignition field pointed to by the source GUID in the rendering parameters template.
        - The property name should match the field name in the rendering parameters template. If they are different (ie. spacing – “Heading Position”), you will need to decorate the parameter.

            - [SitecoreField(FieldName=”Heading Position”)]

    - Set the TemplateId GUID string to match your parameters template item id in Sitecore.
    - Be sure to set AutoMap = true so your model populates correctly.

 ::

    using Glass.Mapper.Sc.Configuration.Attributes;
    using Ignition.Core.Models.BaseModels;
    using Ignition.Core.Models.Settings;

    namespace Ignition.Sc.Components.ContentBlurb
    {
        [SitecoreType(TemplateId = "{153E726E-0100-469F-9B8C-B616FD7CF842}", AutoMap = true)]
        public interface IContentBlurbParams : IParamsBase
        {
            IStringSetting HeadingPosition { get; set; }
        }
    }

Step 5: Update the View Model
    - Update your view model “ContentBlurbViewModel”.
    - Add the “HeadingPosition” property.

        - Notice I’ve used a string (not an IStringSetting).  We will use “IStringSetting.StringSetting” to return a string in our agent.

:: 

    using Ignition.Core.Mvc;
    using Ignition.Sc.Templates;

    namespace Ignition.Sc.Components.ContentBlurb
    {
        public class ContentBlurbViewModel : BaseViewModel
        {
            public IContentBlurb ContentBlurb { get; set; }
            public string HeadingPosition { get; set; }
        }
    }

Step 6: Update the Agent
    - Update your agent “ContentBlurbAgent”.
    - The IParamsBase contains a “RenderingParameters” property that is populated by Ignition.  We will cast it as our “IContentBlurbParams” to access our properties.
    - Assign the parameters value to our view model’s property.

 ::

    using System;
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

                var parameters = RenderingParameters as IContentBlurbParams;
                ViewModel.HeadingPosition = parameters?.HeadingPosition?.StringSetting ?? String.Empty;
            }
        }
    }

Step 7: Update the Controller
    - Update your “ContentBlurbController”.
    - We change our call from View<TAgent, TViewModel> to View<TAgent, TViewModel, TViewParams>() passing the name of our parameters template along with our agent and view model.

 ::

    using System.Web.Mvc;
    using Ignition.Core.Mvc;

    namespace Ignition.Sc.Components.ContentBlurb
    {
        public class ContentBlurbController : IgnitionController
        {
            public ActionResult ContentBlurb()
            {
                return View<ContentBlurbAgent, ContentBlurbViewModel, IContentBlurbParams>();
            }
        }
    }

Step 8: Update the View
    - Update your view “ContentBlurb”.
    - Use the view parameter as needed.
    
        - In this example, I’ve used the string value as a css class name.

:: 

    @model Ignition.Sc.Components.ContentBlurb.ContentBlurbViewModel

    <div class="row contentblurb">
        <div class="col-lg-4 @Model.HeadingPosition">
            <h1>@Html.Glass().Editable(a => a.Heading.Heading, null, true)</h1>
            <h4>@Html.Glass().Editable(a => a.Subtitle.Subtitle, null, true)</h4>
        </div>
        <div class="col-lg-8">
            @Html.Glass().Editable(a => a.Content.RichText1, null, true)
        </div>
    </div>

Step 9: Putting it all together
    - Build and deploy your solution.
    - Edit an instance of the ContentBlurb component.
    - Notice the CSS Setting section with the HeadingPosition dropdown.
    - Make a selection in the dropdown.
    - Publish changes in Sitecore.

The drop down shows the options created in step 1.

.. image:: images\ViewParameter-Dropdown.png

The heading is displayed to the right of the rich content.

.. image:: images\ViewParameter-ViewRight.png