# Set DataGrid Page

https://github.com/stadium-software/datagrid-set-page/assets/2085324/d5dd57c9-f972-491a-a66a-e33eec752394

## Sample applications
This repo contains one Stadium 6.7 application
[SetDataGridPage.sapz](Stadium6/SetDataGridPage.sapz?raw=true)

## Contents
1. SetDataGridPage: A script that you can use to load a specific DataGrid page
2. GetDataGridSelectedPage: A script that returns the currently selected page of a DataGrid

# Version 
1.1 Added logic to detect uniqueness of DataGrid class on page

## Application Setup
Check the *Enable Style Sheet* checkbox in the application properties

## Database, Connector and DataGrid

Use the instructions from [this repo](https://github.com/stadium-software/samples-database) to setup the database and DataGrids for this sample

## SetDataGridPage Global Script Setup

1. Create a Global Script and call it "SetDataGridPage"
2. Add the input parameters below to the script
   1. DataGridClass
   2. PageNo
3. Drag a Javascript action into the script and paste the Javascript below unaltered into the action
```javascript
/* Stadium Script Version 1.1 https://github.com/stadium-software/datagrid-set-page */
let dgClassName = "." + ~.Parameters.Input.DataGridClass;
let dg = document.querySelectorAll(dgClassName);
if (dg.length == 0) {
    dg = document.querySelector(".data-grid-container");
} else if (dg.length > 1) {
    console.error("The class '" + dgClassName + "' is assigned to multiple DataGrids. DataGrids using this script must have unique classnames");
    return false;
} else { 
    dg = dg[0];
}
let table = dg.querySelector("table");
let target = ~.Parameters.Input.PageNo;
let current = table.querySelector(".pagination .active a").textContent;

let steps = calculateSteps(current, target);

let options = {
        childList: true,
        subtree: true,
    },
    observer = new MutationObserver(navigateToPage);

function navigateToPage() {
    observer.disconnect();
    let next = table.querySelector(".pagination li:last-child a");
    let previous = table.querySelector(".pagination li:first-child a");
    if (steps > 0 && next) {
        next.click();
        steps--;
        observer.observe(table, options);
    } else if (steps < 0 && previous) {
        previous.click();
        steps++;
        observer.observe(table, options);
     } else {
        let arr = document.querySelectorAll(".pagination li a");
        for (let i = 0; i < arr.length; i++) {
            if (arr[i].textContent == target) {
                arr[i].click();
            }
        }
    }
}
function calculateSteps(c, t) {
    t = Math.ceil(t / 10) * 10;
    c = Math.ceil(c / 10) * 10;
    let x = t - c;
    return x / 10;
}
navigateToPage();
observer.observe(table, options);
```

## Page Setup

1. Drag a *DataGrid* control to the page (if you don't already have one)
2. Add a class to identify the DataGrid *Classes* property that uniquely identifies this DataGrid on this page (e.g. datagrid-page-navigate)
3. Note: If multiple editable DataGrids are shown on one page, each DataGrid must have a unique classname

## Loading a specific DataGrid Page

1. Drag the "SetDataGridPage" script into any event handler
2. Complete the input parameters
   1. DataGridClass: The unique class you assigned to the DataGrid (e.g. datagrid-page-navigate)
   2. PageNo: The page you wish to display in the DataGrid

## GetDataGridSelectedPage Global Script Setup

1. Create a Global Script and call it "GetDataGridSelectedPage"
2. Add the input parameters below to the script
   1. DataGridClass
3. Add the output parameters below to the script
   1. CurrentPageNo
4. Drag a Javascript action into the script and paste the Javascript below unaltered into the action
```javascript
/* Stadium Script Version 1.1 https://github.com/stadium-software/datagrid-set-page */
let dgClassName = "." + ~.Parameters.Input.DataGridClass;
let dg = document.querySelectorAll(dgClassName);
if (dg.length == 0) {
    dg = document.querySelector(".data-grid-container");
} else if (dg.length > 1) {
    console.error("The class '" + dgClassName + "' is assigned to multiple DataGrids. DataGrids using this script must have unique classnames");
    return false;
} else { 
    dg = dg[0];
}
let table = dg.querySelector("table");
return table.querySelector(".pagination .active a").textContent;
```
1. Drag a *SetValue* action below the *Javascript* action and set the properties
   1. Target: Select the *CurrentPageNo* in the *ScriptOutputParameters* category in the property dropdown
   2. Value: The return value from the JavaScript action (= ~.JavaScript)

![](images/GetDataGridSelectedPageSetValue.png)

## Reloading the current DataGrid page

1. In an Event Handler, first drag in the *GetDataGridSelectedPage* script
2. Enter the *DataGridClass* in the corresponding input parameter (e.g. datagrid-page-navigate)
3. Fetch the data (e.g. execute the query or call an API)
4. Use a *SetValue* to assign the data to the Datagrid
5. Drag the *SetDataGridPage* script under the *SetValue*
6. Enter the input parameters
   1. DataGridClass: The class you assigned to the DataGrid (e.g. datagrid-page-navigate)
   2. PageNo: The ouput of the *GetDataGridSelectedPage* script
