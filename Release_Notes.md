
## Changes

On it's official website you can [download](https://www.mustangproject.org/files/Mustang-CLI-2.0.0-alpha3.jar) a alpha release of Mustang 2.

It integrates the successor of the ZUGFeRD [Validator ZUV](https://github.com/ZUGFeRD/ZUV/) in it's command line tool and can be used as library.

Additionally, the default changed from ZUGFeRD 1 to ZUGFeRD 2.1.1 (previously you had to enable that, now you have to specify that you want to use ZF1 if that's the case), it's now available via Maven Central and additionally to the old Interface-style  pullProvider requiring you to implement methods there is now also a "normal", class-orientierted, halfway fluent "Push-Provider". It's describe below, if you are impatient please feel free to have a look at it's tests on https://github.com/ZUGFeRD/mustangproject/blob/master/library/src/test/java/org/mustangproject/ZUGFeRD/ZF2PushTest.java.
 
This is a preview release, please do not (yet) use it in production.

### Use on command line
`java -jar Mustang-CLI-2.0.0-alpha3.jar --action=combine` embedds a XML into a PDF (A-1) file and exports as PDF/A-3

`java -jar mustang-cli.jar --action=extract` extracts XML from a ZUGFeRD file and

`java -jar mustang-cli.jar --action=validate` validates XML or PDF files.

`java -jar mustang-cli.jar --help` still outputs the parameters which can be used
to for non-interactive (i.e., batch) processing. 


The source file parameter for validation changed 
from `-f` (ZUV) to the usual `--source`. The following 
result codes apply:

| section  | meaning  |
|---|---|
| 1  | file not found  |
| 2  | additional data schema validation fails  |
| 3  | xml data not found  |
| 4  | schematron rule failed  |
| 5  | file too small  |
| 6  | VeraPDFException |
| 7  | IOException PDF  |
| 8  | File does not look like PDF nor XML (contains neither %PDF nor <?xml)  |
| 9  | IOException XML  |
| 11  | XMP Metadata: ConformanceLevel not found  |
| 12  | XMP Metadata: ConformanceLevel contains invalid value  |
| 13  | XMP Metadata: DocumentType not found  |
| 14  | XMP Metadata: DocumentType invalid  |
| 15  | XMP Metadata: Version not found  |
| 16  | XMP Metadata: Version contains invalid value  |
| 18  | schema validation failed  |
| 19  | XMP Metadata: DocumentFileName contains invalid value  |
| 20  | not a pdf  |
| 21  | XMP Metadata: DocumentFileName not found")  |
| 22  | generic XML validation exception  |
| 23  | Not a PDF/A-3  |
| 24  | Issues in CEN EN16931 Schematron Check |
| 25  | Unsupported profile type  |
| 26  | No rules matched, XML to minimal?  |
| 27  | XRechnung Schematron Check |
 
 
### Use as Library

We're now on maven central, please remove the old github repository. Additionally, the following changed

| What  | old value | new value |
|---|---|---|
| Group id  | org.mustangproject.zugferd | org.mustangproject|
| Artifact ID | mustang | library  |
| Version | 1.7.8 | 2.0.0-alpha3  |

If you want you can also embed the validator in your software using validator
as artifact ID. "validator" includes the library functionality but is >20 MB 
bigger due to it's dependencies. 


### Update from 1.x to 2.0

ZF2 was possible with Mustang 1 but it is default in Mustang 2, so 
you will need to `.setZUGFeRDVersion(1)` if you don't want ZUGFeRD 2 files.
`PDFattachZugferdFile` is now called `setTransaction` and instead of
a `ZUGFeRDExporterFromA1Factory` the `ZUGFeRDExporterFromA1` will now return a
a class implementing `IZUGFeRDExporter` instead of a `ZUGFeRDExporter`.
So 
```
			 ZUGFeRDExporter ze = new ZUGFeRDExporterFromA1Factory().setZUGFeRDConformanceLevel(ZUGFeRDConformanceLevel.COMFORT).load(SOURCE_PDF)) {

``` 
changes to 
```
			 IZUGFeRDExporter ze = new ZUGFeRDExporterFromA1().setZUGFeRDVersion(1).setZUGFeRDConformanceLevel(ZUGFeRDConformanceLevel.EN16931).load(SOURCE_PDF)) {

```

The old Contact class has been corrected to TradeParty. The TradeParty class
can now refer to a (human) Contact from the new Contact() class. 

The importer can still be used like

```
ZUGFeRDImporter zi = new ZUGFeRDImporter(inputStream);
String amount = zi.getAmount();
``` 
but there is also the new invoiceImporter 
```

		ZUGFeRDInvoiceImporter zii=new ZUGFeRDInvoiceImporter(TARGET_PDF);

		Invoice invoice=null;
		try {
			invoice=zii.extractInvoice();
		} catch (XPathExpressionException | ParseException e) {
// handle Exceptions
		}
		assertFalse(hasExceptions);
		// Reading ZUGFeRD
		assertEquals("Bei Spiel GmbH", invoice.getOwnOrganisationName());
		assertEquals(3, invoice.getZFItems().length);
		assertEquals("400.0000", invoice.getZFItems()[1].getQuantity().toString());

		assertEquals("160.0000", invoice.getZFItems()[0].getPrice().toString());
		assertEquals("Heiße Luft pro Liter", invoice.getZFItems()[2].getProduct().getName());
		assertEquals("LTR", invoice.getZFItems()[2].getProduct().getUnit());
		assertEquals("7.00", invoice.getZFItems()[0].getProduct().getVATPercent().toString());
		assertEquals("RE-20170509/505", invoice.getNumber());

		SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd");
		assertEquals("2017-05-09",sdf.format(invoice.getIssueDate()));

		assertEquals("Bahnstr. 42", invoice.getRecipient().getStreet());
		assertEquals("88802", invoice.getRecipient().getZIP());
		assertEquals("DE", invoice.getRecipient().getCountry());
		assertEquals("Spielkreis", invoice.getRecipient().getLocation());

		TransactionCalculator tc=new TransactionCalculator(invoice);
		assertEquals(new BigDecimal("571.040000"),tc.getTotalGross());

``` 

### Using the library to create e-invoices
From maven central fetch
```

<dependencies>
    <dependency>
       <groupId>org.mustangproject</groupId>
       <artifactId>library</artifactId>
       <version>2.0.0-alpha3</version>
    </dependency>
</dependencies>

```
```
import org.mustangproject.Contact;
import org.mustangproject.Invoice;
import org.mustangproject.Item;
import org.mustangproject.Product;
import org.mustangproject.ZUGFeRD.IZUGFeRDExporter;
import org.mustangproject.ZUGFeRD.Profiles;
import org.mustangproject.ZUGFeRD.ZUGFeRDExporterFromA1;
import java.math.BigDecimal;
import java.util.Date;

public class Main {
    public static void main(String[] args) {
    Invoice i = new Invoice().setDueDate(new Date()).setIssueDate(new Date()).setDeliveryDate(new Date()).setOwnOrganisationName("My company").setOwnStreet("teststr").setOwnZIP("12345").setOwnLocation("teststadt").setOwnCountry("DE").setOwnTaxID("4711").setOwnVATID("0815").setRecipient(new Contact("Franz Müller", "0177123456", "fmueller@test.com", "teststr.12", "55232", "Entenhausen", "DE")).setNumber("INV/123").addItem(new Item(new Product("Testprodukt", "", "C62", new BigDecimal(0)), new BigDecimal(1.0), new BigDecimal(1.0)));
        try {
            IZUGFeRDExporter ie = new ZUGFeRDExporterFromA1().load("source.pdf").setZUGFeRDVersion(2).setProfile(Profiles.EN16931);
            ie.setProfile(Profiles.EN16931).setTransaction(i).export("target.pdf");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### Export XML
A invoice like this
```
Invoice i = new Invoice().setDueDate(new Date()).setIssueDate(new Date()).setDeliveryDate(new Date()).setOwnOrganisationName(orgname).setOwnStreet("teststr").setOwnZIP("55232").setOwnLocation("teststadt").setOwnCountry("DE").setOwnTaxID("4711").setOwnVATID("0815").setRecipient(new Contact("Franz Müller", "0177123456", "fmueller@test.com", "teststr.12", "55232", "Entenhausen", "DE")).setNumber(number).addItem(new Item(new Product("Testprodukt", "", "C62", new BigDecimal(0)), amount, new BigDecimal(1.0)));
```
can be used to get XML
```
ZUGFeRD2PullProvider zf2p = new ZUGFeRD2PullProvider();
zf2p.generateXML(i);
String theXML = new String(zf2p.getXML());
```
or can also be used with setTransaction to generate invoice PDFs straight away.
### Embedding ZF1 to ZF2 migration
The functionality is based on the XSLT file in library/src/main/resources/stylesheets/ZF1ToZF2.xsl,
it can be accessed via 
```
		XMLUpgrader zmi = new XMLUpgrader();
		String xml = zmi.migrateFromV1ToV2(xmlName);
		Files.write(Paths.get(outName), xml.getBytes());

```
### Embedding ZF2 visualization 

In case you don't want to access this functionality over the 
commandline you can use
```
		ZUGFeRDVisualizer zvi = new ZUGFeRDVisualizer();
			xml = zvi.visualize(sourceName);
			Files.write(Paths.get("factur-x.xml"), xml.getBytes());
```
for the visualizer.     The output requires a CSS and a javascript file which are in the
	    jar's resources as 
		(/src/main/resources/)xrechnung-viewer.css
		(/src/main/respurces/)xrechnung-viewer.js


### Embedding the validator
The validator library also contains the functionality to 
read/write ZUGFeRD-invoices like the (smaller) library module. 
```
<dependencies>
   <dependency>
      <groupId>org.mustangproject</groupId>
      <artifactId>validator</artifactId>
      <version>2.0.0-alpha3</version>
   </dependency>
</dependencies>

```
```
import org.mustangproject.validator.ZUGFeRDValidator;
public class Main {
    public static void main(String[] args) {
        ZUGFeRDValidator zfv = new ZUGFeRDValidator();
        System.out.println(zfv.validate("/tmp/factur-x.xml"));
    }
}
```
