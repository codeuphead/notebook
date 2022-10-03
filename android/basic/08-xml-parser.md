# Xml解析

## Pull

```java
XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
XmlPullParser xmlPullParser = factory.newPullParser();
xmlPullParser.setInput(new StringReader(xmlStringData));
int eventType = xmlPullParser.getEventType();
while (eventType != xmlPullParser.END_DOCUMENT) {
    String nodeName = xmlPullParser.getName();
    switch(eventType){
        case XmlPullParser.START_TAG:
            if("id".equals(nodeName)){
                String id = xmlPullParser.nextText();
            }else if(){

            }
        break;
        case XmlPullParser.END_TAG:
        //解析完某个标签
        break;

    }
    eventType = xmlPullParser.next();
}
```

## SAX

通常创建一个 DefaultHandler 的子类，重写父类5个方法。

```java
public class SAXHandler extends DefaultHandler {
    @Override
    public void startDocument() throws SAXException {

    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException{

    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException{

    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {

    }

    @Override
    public coid endDocument() throws SAXException {

    }
}
```

```java
SAXParserFactory factory = SAXParserFactory.newInstance();
XMLReader reader = factory.newSAXParser().getXMLReader();
ContentHandler handler = new ContentHandler();
reader.setContentHandler(handler);
reader.parser(new InputSource(new StringReader(xmlStringData)));
```

- characters 方法可能会调用多次，一些黄行副也被当作内容解析出来，因此通常将 char[] append 到 StringBuilder 上，并且最后调用 trim()，解析完一个节点，清空 StringBuilder

## DOM

一次将所有内容都加载到内存中