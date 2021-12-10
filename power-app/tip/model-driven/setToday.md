function SetToday(date)
{    
var isCreateForm = Xrm.Page.ui.getFormType() == 1;
var dateField = Xrm.Page.getAttribute(date);
    if (isCreateForm)   // Check that this is a new Record
    {        dateField.setValue(new Date()); // Set the Date field to Today
             dateField.setSubmitMode("always");   // Save Disabled Fields
    }
}