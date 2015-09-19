# Description
This module creates a table for any kind of data for you in a resource view. 

In combination with the [form module](./pong-form/) you have a powerful GUI for RESTfule web service out there.

# Usage in "structure"

Simply <code>"type": "pong-table"</code> to the <code>rows</code> or <code>cols</code> resource. Example [[PoNG Structure Specification|structure file]] extract:

	 {
	   "layout": {
	      ...
	      "rows": [
	      {
	        "rowId": "bla",
	        "resourceURL": "Products",
	        '''"type": "pong-table"''',
	        ...
	      },
	      ...
	    ],
	    ...
	 }

## Definition of the Table 
The resource will load the table definition from the URL <code>../svc/XYZ/pong-table</code>

Example JSON definiton from <code><nowiki>../svc/[resourceUrl]/pong-table</nowiki></code>
 
	 {
	     "dataURL": "webdata",
	     "filter" : {
	         "dataReqParams": [ {"id":"productSearchString", "label":"Product"}, {"id":"productType", "label":"Type"} ],
	         "dataReqParamsSrc": "Form"
	     },
	     "rowId": "productId",
	     "cols" : [, 
	        { "id": "name", "label": "Name", "cellType": "text", "editable":"true", "width":"20%" }, 
	        { "id": "descr", "label": "Description", "cellType": "text", "width":"40%" },  
	        { "id": "img", "label": "Picture", "cellType": "img", "width":"20%" },  
	        { "id": "addToOrder", "label": "Add", "cellType": "button", "width":"20%" } 
	    ],
	    "maxRows":"10"
	 }

The <code>"filter"</code> is optional. 

In the example the table is the top level object in the content. It may happen, that the table is somewhere in the object tree of the JSON result. You can use the dataDocSubPath attribute, to tell  the module the sub-path of the table data, e.g.

	{
		"dataURL": "webdata",
		"dataDocSubPath": "searchResult.dataTbl",
		...
    }

The identifier of each row is defined by <code>"rowId"</code>. Thi can be a string as above or a list of strings (e.g. when the table data is a join of two tables). Example:

	"rowId": ["struct_id","template_id"]
	
The rowId key and values are used as GET or POST prameters in updates of editable cells or for button actions.

Remark: The rowId must not be in the cols array, it must only be in the result of the service call.

## Columns 
### Cell Types 
<code>"cellType"</code> property can be:
* text 
* checkbox
* img
* link (TODO)
* linkLink
* button
* tooltip
* rating

The img type and link type will expect a URL as content.

### linkLink 
Generates a link with "label" as link text and "id" is set as href URL.

Optional attributes:
* "target", e.g. "_blank"
* "URL" recommended to use, "rowId" value will be used as GET parameter, if the URL already contains a GET parameter, the "rowId" will be appended

Example:

	 "rowId":"data_id",
	 "cols":[
	     { "id":"mylink", "label":"Text", "cellType":"linkLink", "URL":"other-page.php", "target":"_blank", "width":"5%" },
	     ...
	 ]	

will generate a column:

	 <nowiki>
	 <td>
	 <a href="other-page.php?data_id=1243>Text</a>
	 </td>
	 </nowiki>

### Button 
The button will do an asynchronous call to <code><nowiki>svc/[resourceURL]/<id of button>?rowId=[rowId]</nowiki></code> 
In the example above this will be posted (productId of the row may be 1234): <code>svc/Products/addToOrder?rowId=1234</code>

You can set the <code>URL</code> parameter to any other AJAX target.

Button <code>method</code> is optional, default is <code>"method":"POST"</code> 

After pressing the button, the data in the table is updated.

Optional parameters (for actions after a successful POST call):
* <code>"update"</code> is an (optional) array of resource (column/row) ids, where further data updates should be triggered. Example: <code>"update": [ { "resId":"xyz" } ] }</code>
* <code>"target"</code> 
** target can be a resource Id: <code>"target": "customerActionOut"</code>
** target can be "_parent", the result of the AJAX call must be an URL
** target may me "modal", to show the call result in a modal dialog
* <code>"setData"</code>: define array of <code>resId</code> and optional <code>dataDocSubPath</code> 	 
For POST; GET, DELETE methods the rowId(s) are used as params to identify the row of the table.

#### Example:

	 "rowId": "id",
	 "cols" : [ 
	    { "id":"name", "label":"Name", "cellType": "text", "width":"80%" },
	    { "id":"Btn", "label":"Load", "cellType":"button", "URL":"/service", "width":"20%", "update": [ { "resId":"Form" } ] } 
	 ]
	 
If you press the "Load" button then it will do:

<code>POST /service<br>
POST param: { "id":"123" }</code>

call update JS of aForm, which results in <code>GET /<ressource-URL-of-aForm>?id=123</code>

#### Create Custom Data For Button-Request 
A practical option is to form a completely new request and merge data of the table row into the request.
The <code>params</code> array must define the new custom parameters <code>key</code> and <code>value</code>. 
To reuse values of the current table row, placeholders can be specified by <code>${columnname}</code>.
 
Practical example passing data from SugarCRM to OpenStreeMap Nominatim search:

	{
	 "dataURL": "tbldata", 
	 "rowId": "id", 
	 "cols" : [ 
	   { "id": "name", "label": "Name", "cellType": "text", "width":"20%" }, 
	   { "id": "map", "label": "Map", 
	     "cellType": "button", 
	     "URL":"http://open.mapquestapi.com/nominatim/v1/search.php", 
	     "method":"GET",
	     "params":[ 
	         { "name":"key", "value":"custome-rsecret" }, 
	         { "name":"format", "value":"json" }, 
	         { "name":"city", "value":"${primary_address_city}" }, 
	         { "name":"street", "value":"${primary_address_street}" }, 
	         { "name":"country", "value":"${primary_address_country}" }, 
	         { "name":"addressdetails", "value":"1" }, 
	         { "name":"limit", "value":"5" }
	      ], 
	     "setData":[ {"resId":"osmSearchResult"} ],
	     "width":"5%" 
	   }
	 ],  
	 "maxRows":"10"
	}

### Data loading 
The data will be requested from the URL <code><nowiki>svc/[resourceURL]/[dataURL]?productSearchString=[productSearchString]&productType=[productType>]</nowiki></code>. 
The <code>dataReqParams</code> will be looked up in the session map. You can use e.g. [[PoNG Module: Form View]] to set these values in the session map.

Options for <code>dataReqParams</code>:
* Form
* SessionMap (only prepared in v0.1)

### Tooltip 
Special definitions for "cols":
* tooltip (label is the id of the column, where tool tip should be applied)

Tooltip does not render an individual column.

Example:

	 "cols" : [ 
	    { "id":"name", "label":"Type", "cellType": "text", "width":"80%" },
	    { "id":"descr", "label":"name",  "cellType":"tooltip" },
	    ...
	 ]


### IMG Actions 
GET/POST example:

	 "action" : { "method":"GET" ,"actionURL": "svc/customer", "update": "customerForm" }
	update (=module ID) is optional
	 "action" : { "method":"POST" ,"actionURL": "svc/customer" }

Link example:

	"action" : { "method":"GET" ,"actionURL": "svc/customerHTML", "target": "customerActionOut" }

Function call example:

	"action" : { "method":"JS" ,"actionURL": "updateCustomer($id)", "update": "customerForm" }
	
where id is a table column

### Editable Cells 
If you specify <code>"editable":"true"</code> for a column, you can click into the cell and edit. When you leave the focus of the cell, the cell data and the "rowId" and its ID value are JSON encoded and POSTed back to the resource, to store them modified. Example HTTP request to save the modified cell data of cell column "name": 

	<code>POST /svc/Products/webdata { "productId": "03", "name": "Hello" }</code>

Currently this feature is only available for text and checkbox columns.

'''Important''': The URL ending is a diva. If you implement a service endpoint (e.g. using <code>svc/myservice/index.php</code>) the URL may have to end with a slash. If you specify <code>"resourceUrl":"svc/myservice"</code> you can use <code>"dataURL":"/"</code> to get the POST request working correctly. The "developer tools" are your friend to identify, what request is going out -- and if it works.

### Rating Column Type
The rating expects a typically a number as result, but can be anything. 
You have to define a rating Type, e.g. <code>"ratingType":"3star"</code>. 
The logic is to load a image by appending the rating value to the rating type and append ".png".

Currently the table module supports:
* <code>"ratingType":"3star"<code>: rating values= [ "0", "1", "2", "3" ]</code>
* <code>"ratingType":"5star"<code>:rating values= [ "0", "1", "2", "3", "4", "5" ]</code>
* <code>"ratingType":"prio"<code>: rating values= [ "0", "1", "2", "3" ]</code>

## Simple example
	{
	    "dataURL": "webdata",
	    "dataReqParams": [ {"id":"productSearchString", "label":"Product"}, {"id":"productType", "label":"Type"} ],
	    "dataReqParamsSrc": "Form",
	    "dataDocSubPath": "struc1.tbl",
	    "rowId": "productId",
	        "cols" : [
	           { "id": "name", "label": "Name", "cellType": "text" }, 
	           { "id": "link", "label": "Data Sheet", "cellType": "linkLink" }, 
	           { "id": "descr", "label": "name", "cellType": "tooltip" },  
	           { "id": "img", "label": "Picture", "cellType": "img" },  
               { "id": "rating", "label":"Rating", "cellType":"rating", "ratingType":"3star" },
	           { "id": "order", "label": "Add", "cellType": "button" }  
	        ],         
	        "maxRows":"5"
	}

And a HTTP GET to webdata gives

	{
	    "bla": "blub",
	    "struc1": {
	        "tbl": [
	            {"productId":"xxx1","name":"xxx1","link":"http://bla/xxx.pdf","descr":"Blah blub bubber.","img":"img/x01.png"},
	            {"productId":"yyy2","name":"yyy2":"http://bla/yyy.pdf","descr":"Blah blub bubber.","img":"img/x02.png"},
	            ...
	            {"productId":"15","name":"yy15","link":"http://bla\yyy.pdf","descr":"Blah blub bubber.","img":"img/x15.png"}
	        ]
	    }
	}

## Table CSS 
You can use the following CCS elements to define thy style of the table:
* Table ID: #<resId>ContentPongTable
* Table class: pongTable
* optional Form DIV ID: #<resId>ContentPongTableFrm
* optional From DIV class: pongTableFrm