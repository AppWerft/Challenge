# Challenge

<img src="https://www.sencha.com/img/sencha-large.png" width="200" /> 

## Import einer CSV in Sencha extJS

Daten werden im Sencha Sytem intern in einem Model bzw. Store gehalten. Für den Import externer Daten wird ein Proxy benötigt, der einen Reader nutzt. CSV gehört nicht zur Werkzeugkiste, sondern nur JSON, XML und Array. Als `reader` muss also eine neue Klasse eingesetzt werden, die sich von `Ext.data.reader.JSON` ableitet:
```javascript
store: new Ext.data.Store({
     model: 'myProject',
     proxy: {
       type: 'ajax',
       url: 'http://csv-uri/fake.txt',
       reader:  Ext.create('CsvReader'); 
     },
}),
```
### CsvReader

Vom Grundsatz her wird die *class*  _Ext.data.reader.Json_ mit neuen angepassten Methoden überschrieben. Die Magie steckt in der Methode _toJson()_ 

```javascript
Ext.define('CsvReader', {
    extend: 'Ext.data.reader.Json',
    alias : 'reader.csv',
    toJson: function(csvData){
      // Holla die Waldfeee
      return [];
    },
 
    // override, nett hier rein textuell, oder doch Teil des Compilers?
    getResponseData: function(response) {
        try {
            return this.readRecords(response.responseText);
        } catch (ex) {
            error = new Ext.data.ResultSet({
                total  : 0,
                count  : 0,
                records: [],
                success: false,
                message: ex.message
            });
            this.fireEvent('exception', this, response, error);
            return error;
        }
    },
 
    // override
    readRecords: function(strData) {
        var result = this.toJson(strData);
        return this.callParent([result]);
    }
});
```
Nun muss also neuer Code in das Projekt. Die Qualität dieses Readers kann nicht die Stabilität des Gesamtprojektes „runterziehen“. Beispielsweise ist die Lösung wie in [fahd.blog](http://fahdshariff.blogspot.de/2014/09/ext-js-csv-data-reader.html) sehr gefährlich, da sie viele Dinge unterstellt:
- [x] Datei ist im „richtigen“ Format,
- [x] Datei ist klein genug, um in den Speicher zu passen,
- [x] CSV ist genau in dem Format, also kommasepariert und in Doppelquote eingepackt,
- [x] unterstellt, dass in den Datensätzen keine Zeilenumbrüche sind.

Es muss eine robuste, verlässliche Library her. Ohne das wirklich geprüft zu haben, könnte [Papa Parse](http://papaparse.com/) eine Lösung sein. 

Um so weit wie möglich in der Senchawelt zu bleiben, wird es sinnvolls ein, die Funktion:
```javascript
toJson: function(csvData){
      var result = Papa.parse(csvData[, config]);
      if (Array.isArrqay(result))
          return result;
    },  // eventuell Anbinddung an Error-System …

```
zu nutzen, weil man mit der String-Schnittstelle im Schema bleibt.  Die Konfiguration des Parsers ist [hier](http://papaparse.com/docs#config), Für sehr große CSV-Dateien bräucht man ein System, das in Abhängigkeit des Viewports in der CSV-Datei seekt (RecyclerView in Android)

## Kundengerechte Darstellung eines Projektverlaufes 

Für die Darstellung des Projektverlaufes bietet sich ein Gantt-Diagramm an. Intern wird für die Darstellung von Charts die Klasse `Ext.chart` verwendet, Leider ist Gantt nicht Bestandteil dieser Klasse . 

Nun gibt es zwei zielführende Wege:

### Eigenentwickelte Chart

Bei diesem Weg wird aus Primitivelementen eine neue Chart entwickelt. Das ist [hier im Sencha Blog](https://www.sencha.com/blog/create-custom-charts-using-sencha-charts/) erläutert. Diese Lösung ist zwar kompromisslos individuell, aber sich auch die mit höherem Aufwand.

### Nutzung einer Fremdbibliothek
Mit dem Kontrukt `Ext.Component` können Fremdbibliotkeken eingebunden werden. Die Herausforderung wird sein, die entsprechenden Schnittstellen für das Data Binding usw, zu schaffen. Vorteil ist die große Anzahl von fertigen Möglichkeiten, Nachteil die Beschränkung auf die fertige (konfigurierbare) Funktionalität.

Außerhalb des Sencha-Welt hätte man die Auswahl in über 50 verschiedenen Chart-Libraries, beispielsweise [HighChart](https://www.highcharts.com/demo). Welche Lösung gewinnt hängt selbstverständlich von den Kundenanforderungen ab.

![](http://www.aceproject.com/img/ss/gantt-chart-en.jpg)

Highcharts unterstützt so wie Sencha von Haus aus keine Gantt-Diagramme. Das ist die Challenge ;-)

Nun ist [Rapha&euml;l](https://github.com/DmitryBaranovskiy/raphael) Teil von Sencha. Eine schlichte Lösung (ohne Interaktion) wäre von [Erikthered](https://github.com/erikthered/raphael-gantt) 
![](https://raw.githubusercontent.com/erikthered/raphael-gantt/master/examples/example.png)

In Abhängigkeit vom Budget und Zeit sit alles möglich, eventuell auch diese Lösung:

![](https://d2myx53yhj7u4b.cloudfront.net/sites/default/files/online-gantt-chart.gif)

[Quelle](https://www.smartsheet.com/s/global-online-gantt-chart?s=156&c=73&m=414&a=51794163427&k=%2Bgantt&mtp=b&adp=1t2&net=g&dev=c&devm=&mkwid=sDQm8pEhv|dc&plc=&gclid=Cj0KEQjwi7vIBRDpo9W8y7Ct6ZcBEiQA1CwV2BLs3E9Clqc4Ct6wDCRY8pHtgDLJWxKeOreA62eliJsaAn6e8P8HAQ)

Hier könnte in die Herausforderung in der Anbindung bestehen.
