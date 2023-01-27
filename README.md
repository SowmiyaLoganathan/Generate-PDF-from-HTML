# Generate PDF from HTML template 
This repository contains examples to generate dynamic PDF documents from various HTML templates using [Syncfusion .NET HTML to PDF converter library](https://www.syncfusion.com/document-processing/pdf-framework/net/html-to-pdf) in C#. 

The HTML-to-PDF converter is a .NET library for converting webpages, SVG, MHTML, and HTML files to PDF using C#. It uses the popular rendering engine Blink (Google Chrome). It is reliable and accurate. The result preserves all graphics, images, text, fonts, and the layout of the original HTML document or webpage. So you can easily style your PDF reports by updating the CSS file with your images and fonts in C#. 

## Getting Started 
Syncfusion PDF generation allows you to create a invoice PDF document from a HTML template with use of all the tools as like below: 
* CSS for styling 
* JSON for data 
* Fonts 
* Images 

### Create an HTML template 
An HTML template contains placeholders with {{mustache}} syntax. It is used to bind the actual data to the HTML template. For this example, we'll use the [Scriban scripting language](https://github.com/scriban/scriban) to create the placeholders. It's a lightweight scripting language and engine for .NET. 

We design the below HTML template by adding the invoice number, company details and some additional datas. 

```html

<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Invoice</title>
    <link rel="stylesheet" href="style.css" media="all" />
  </head>
  <body>
      <div class="grid-container">
          <div>
              <img src="C:\Users\JeyalakshmiThangamar\Downloads\HtmlToPdfApiTest\HtmlToPdfApiTest\Template\logo.png" style="height: 70px;">
          </div>
          <div style="margin-left:30px; margin-top:20px"><b>{{company_details.name}}</b></div>
      </div>
      <div style="text-align:right;margin-right:10px; margin-top:15px"><b>Invoice No.#23698720</b></div>
      <br />
      <main>
          <table border="0" cellspacing="0" cellpadding="0">
              <thead>
                  <tr>
                      <th class="no">#</th>
                      <th class="desc">DESCRIPTION</th>
                      <th class="unit">UNIT PRICE</th>
                      <th class="qty">QUANTITY</th>
                      <th class="total">TOTAL</th>
                  </tr>
              </thead>
              <tbody id="invoiceItems">
                  {{- index = 1 -}}
                  {{ for item in items }}
                  <tr>
                      <td class="no">{{index}}</td>
                      <td class="desc"><h3>{{item.name}}</h3>{{item.description}}</td>
                      <td class="unit">${{item.price}}</td>
                      <td class="qty">{{item.quantity}}</td>
                      <td class="total">${{item.total_price}}</td>
                  </tr>
                  {{index = index + 1}}
                  {{end}}
              </tbody>
              <tfoot>
                  <tr>
                      <td colspan="2"></td>
                      <td colspan="2">SUBTOTAL</td>
                      <td>${{sub_total}}</td>
                  </tr>
                  <tr>
                      <td colspan="2"></td>
                      <td colspan="2">TAX 25%</td>
                      <td>${{tax}}</td>
                  </tr>
                  <tr>
                      <td colspan="2"></td>
                      <td colspan="2">GRAND TOTAL</td>
                      <td>${{grand_total}}</td>
                  </tr>
              </tfoot>
          </table>
      </main>
  </body>
</html>

```

### Create CSS file 

Cascading Style Sheets (CSS) is a style sheet language used for describing the presentation of a HTML document. Here we have design the style sheet with custom fonts and image for invoice HTML template to apply the styling. 

```css

@font-face {
  font-family: SourceSansPro;
  src: url(SourceSansPro-Regular.ttf);
}
.grid-container {
    display: grid;
    grid-template-columns: 60px auto;
    padding: 10px;
    font-size: 1.2em;
    font-weight: normal;
    text-align: left;
}
table {
  width: 100%;
  border-collapse: collapse;
  border-spacing: 0;
  margin-bottom: 20px;
}
table th,
table td {
  padding: 20px;
  background: #EEEEEE;
  text-align: center;
  border-bottom: 1px solid #FFFFFF;
}
table th {
  white-space: nowrap;        
  font-weight: normal;
}
table td {
  text-align: right;
}
table td h3{
  color: #db3311;
  font-size: 1.2em;
  font-weight: normal;
  margin: 0 0 0.2em 0;
}
table .no {
  color: #FFFFFF;
  font-size: 1.6em;
  background: #db3311;
}
table .desc {
  text-align: left;
}
table .unit {
  background: #DDDDDD;
}
table .qty {
}
table .total {
  background: #db3311;
  color: #FFFFFF;
}
table td.unit,
table td.qty,
table td.total {
  font-size: 1.2em;
}
table tbody tr:last-child td {
  border: none;
}
table tfoot td {
  padding: 10px 20px;
  background: #FFFFFF;
  border-bottom: none;
  font-size: 1.2em;
  white-space: nowrap; 
  border-top: 1px solid #AAAAAA; 
}
table tfoot tr:first-child td {
  border-top: none; 
}
table tfoot tr:last-child td {
  color: #db3311;
  font-size: 1.4em;
  border-top: 1px solid #db3311; 
}
table tfoot tr td:first-child {
  border: none;
}

``` 

The following screenshot shows the output of the HTML template with styled CSS.
<img src="invoiceHTMLTemplate.png" alt="Invoice HTML Template" width="100%" Height="Auto"/>

### Access the data and bind it with the HTML template 

We have used a custom JSON file to store and retrieve the data. The following code snippet is used to create the invoice model.

```csharp

Var invoice= JsonConvert.DeserializeObject<Invoice>(File.ReadAllText("../../../InvoiceData.json"));

```

Now, we use Scriban to bind the data from the model to the HTML template, as explained in the following code example.

```csharp

var sObject = BuildScriptObject(invoice);
var templateCtx = new Scriban.TemplateContext();
templateCtx.PushGlobal(sObject);

var template = Scriban.Template.Parse(pageContent);
var htmlText = template.Render(templateCtx);

```
### Convert HTML string to PDF 

We use the [HtmlToPdfConverter](https://help.syncfusion.com/cr/file-formats/Syncfusion.HtmlConverter.HtmlToPdfConverter.html) to convert the HTML string to a PDF document. Refer to the following code example.

```csharp 

//Initialize HTML to PDF converter with Blink rendering engine.
HtmlToPdfConverter htmlConverter = new HtmlToPdfConverter();

//Convert HTML string to PDF. 
PdfDocument document = htmlConverter.Convert(htmlText , Path.GetFullPath("Template"));

//Save and close the PDF document.
FileStream fs = new FileStream("Output.pdf", FileMode.OpenOrCreate, FileAccess.ReadWrite);
document.Save(fs);
document.Close(true);

```

### Create PDF Generation API 

[Minimal API with ASP.NET Core project](https://learn.microsoft.com/en-us/aspnet/core/tutorials/min-web-api?view=aspnetcore-7.0&tabs=visual-studio) is used to create PDF generation API. In this application, we have used **RestClient** class to sets the localhost IP address and add the additional assets (ie., CSS, fonts, images, PDF size, etc) with your request (**RestRequest**).

```csharp

var client = new RestClient("https://localhost:7094/pdf"); 
var request = new RestRequest().AddFile("index.html", "Template/index.html")
 .AddFile("style.css", "Template/style.css").AddFile("data.json", "Template/InvoiceData.json").AddFile("logo.png", "/logo.png").AddFile("SourceSansPro-Regular.ttf", "Template/SourceSansPro-Regular.ttf");
request.Method = Method.Post; 
var json = new JsonObject
{
    ["index"] = "index.html",
    ["data"] = "data.json",
    ["width"] = 595,
    ["height"] = 842,
    ["margin"] = 0,
    ["assets"] = new JsonArray
 {
 "style.css",
 "SourceSansPro-Regular.ttf",
 "logo.png"
 }
}; 
request.AddParameter("application/json", json.ToString(), ParameterType.RequestBody); 
var response = await client.ExecuteAsync(request);
if (response.StatusCode == System.Net.HttpStatusCode.OK)
{
    System.IO.File.WriteAllBytes("Result.pdf", response.RawBytes);
    
}

``` 

### How to run the examples
* Download this project to a location in your disk. 
* Open the solution files of both projects using Visual Studio. 
* Rebuild the solution to install the required NuGet package. 
* Run the application (PDFFromHTML). Once the browser window opens, then run the API application (HtmlToPdfApiTest). 

# Resources
*   **Product page:** [Syncfusion PDF Framework](https://www.syncfusion.com/document-processing/pdf-framework/net)
*   **Documentation page:** [Syncfusion .NET PDF library](https://help.syncfusion.com/file-formats/pdf/overview)
*   **Online demo:** [Syncfusion .NET PDF library - Online demos](https://ej2.syncfusion.com/aspnetcore/PDF/CompressExistingPDF#/bootstrap5)
*   **Blog:** [Syncfusion .NET PDF library - Blog](https://www.syncfusion.com/blogs/category/pdf)
*   **Knowledge Base:** [Syncfusion .NET PDF library - Knowledge Base](https://www.syncfusion.com/kb/windowsforms/pdf)
*   **EBooks:** [Syncfusion .NET PDF library - EBooks](https://www.syncfusion.com/succinctly-free-ebooks)
*   **FAQ:** [Syncfusion .NET PDF library - FAQ](https://www.syncfusion.com/faq/)

# Support and feedback
*   For any other queries, reach our [Syncfusion support team](https://www.syncfusion.com/support/directtrac/incidents/newincident?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples) or post the queries through the [community forums](https://www.syncfusion.com/forums?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples).
*   Request new feature through [Syncfusion feedback portal](https://www.syncfusion.com/feedback?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples).

# License
This is a commercial product and requires a paid license for possession or use. Syncfusion’s licensed software, including this component, is subject to the terms and conditions of [Syncfusion's EULA](https://www.syncfusion.com/eula/es/?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples). You can purchase a licnense [here](https://www.syncfusion.com/sales/products?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples) or start a free 30-day trial [here](https://www.syncfusion.com/account/manage-trials/start-trials?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples).

# About Syncfusion
Founded in 2001 and headquartered in Research Triangle Park, N.C., Syncfusion has more than 26,000+ customers and more than 1 million users, including large financial institutions, Fortune 500 companies, and global IT consultancies.

Today, we provide 1600+ components and frameworks for web ([Blazor](https://www.syncfusion.com/blazor-components?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [ASP.NET Core](https://www.syncfusion.com/aspnet-core-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [ASP.NET MVC](https://www.syncfusion.com/aspnet-mvc-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [ASP.NET WebForms](https://www.syncfusion.com/jquery/aspnet-webforms-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [JavaScript](https://www.syncfusion.com/javascript-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [Angular](https://www.syncfusion.com/angular-ui-components?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [React](https://www.syncfusion.com/react-ui-components?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [Vue](https://www.syncfusion.com/vue-ui-components?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), and [Flutter](https://www.syncfusion.com/flutter-widgets?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples)), mobile ([Xamarin](https://www.syncfusion.com/xamarin-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [Flutter](https://www.syncfusion.com/flutter-widgets?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [UWP](https://www.syncfusion.com/uwp-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), and [JavaScript](https://www.syncfusion.com/javascript-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples)), and desktop development ([WinForms](https://www.syncfusion.com/winforms-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [WPF](https://www.syncfusion.com/wpf-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [WinUI(Preview)](https://www.syncfusion.com/winui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples), [Flutter](https://www.syncfusion.com/flutter-widgets?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples) and [UWP](https://www.syncfusion.com/uwp-ui-controls?utm_source=github&utm_medium=listing&utm_campaign=github-docio-examples)). We provide ready-to-deploy enterprise software for dashboards, reports, data integration, and big data processing. Many customers have saved millions in licensing fees by deploying our software.











