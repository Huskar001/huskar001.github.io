---
layout: post
title:  "map与xml的相互转换"
date:   2017-07-21 08:55:01 +0800
categories: web
tag: -web
---
* content
{:toc}

最近遇到一个web接口，发送和返回都需要使用xml格式，而map是java比较易用的工具。


maptoxml
-----------
		import org.jdom.*;
		Element root = new Element("advPay");
		 // 根节点添加到文档中；
		 Document Doc = new Document(root);
		 Element pubinfo = new Element("pubinfo");
		 Element busiData = new Element("busiData");
		
		 pubinfo.addContent(new Element("version").setText("xuehui"));
		 pubinfo.addContent(new Element("transactionId").setText("28"));
		 pubinfo.addContent(new Element("transactionDate").setText("28"));
		 pubinfo.addContent(new Element("busiCode").setText("28"));
		 pubinfo.addContent(new Element("originId").setText("28"));
		 pubinfo.addContent(new Element("returnCode").setText("28"));
		 pubinfo.addContent(new Element("returnMsg").setText("28"));
		 pubinfo.addContent(new Element("verifyCode").setText("28"));
		
		 busiData.addContent(new Element("token").setText("token"));
		 busiData.addContent(new Element("productId").setText("28"));
		
		
		 root.addContent(pubinfo);
		 root.addContent(busiData);
		 Doc.setRootElement(root);
		
		 XMLOutputter outputer = new XMLOutputter();
		 outputer.setFormat(Format.getPrettyFormat());
		 String xml = outputer.outputString(Doc);
		 System.out.println(xml);

输出

	<?xml version="1.0" encoding="UTF-8"?>
	<advPay>
		<pubinfo>
			<version>xuehui</version>
			<transactionId>28</transactionId>
			<transactionDate>28</transactionDate>
			<busiCode>28</busiCode>
			<originId>28</originId>
			<returnCode>28</returnCode>
			<returnMsg>28</returnMsg>
			<verifyCode>28</verifyCode>
		</pubinfo>
		<busiData>
			<token>token</token>
			<productId>28</productId>
		</busiData>
	</advPay>
	
xmltomap
------------

xml转换为map分两种情况
1 xml中没有重复元素，只需要取最低层级的key和value
	
		public static Map<String, Object> xml2Map(String xmlStr)
			throws JDOMException, IOException {
		Map<String, Object> rtnMap = new HashMap<String, Object>();
		SAXBuilder builder = new SAXBuilder();
		Document doc = builder.build(new StringReader(xmlStr));
		// 得到根节点
		Element root = doc.getRootElement();
		String rootName = root.getName();
		rtnMap.put("root.name", rootName);
		// 调用递归函数，得到所有最底层元素的名称和值，加入map中
		convert(root, rtnMap, rootName);
		return rtnMap;
	}

	public static void convert(Element e, Map<String, Object> map,
			String lastname) {
		if (e.getAttributes().size() > 0) {
			Iterator it_attr = e.getAttributes().iterator();
			while (it_attr.hasNext()) {
				Attribute attribute = (Attribute) it_attr.next();
				String attrname = attribute.getName();
				String attrvalue = e.getAttributeValue(attrname);
				// map.put( attrname, attrvalue);
				map.put(lastname + "." + attrname, attrvalue); // key 根据根节点 进行生成
			}
		}
		List children = e.getChildren();
		Iterator it = children.iterator();
		while (it.hasNext()) {
			Element child = (Element) it.next();
			/* String name = lastname + "." + child.getName(); */
			String name = child.getName();
			// 如果有子节点，则递归调用
			if (child.getChildren().size() > 0) {
				convert(child, map, lastname + "." + child.getName());
			} else {
				// 如果没有子节点，则把值加入map
				map.put(name, child.getText());
				// 如果该节点有属性，则把所有的属性值也加入map
				if (child.getAttributes().size() > 0) {
					Iterator attr = child.getAttributes().iterator();
					while (attr.hasNext()) {
						Attribute attribute = (Attribute) attr.next();
						String attrname = attribute.getName();
						String attrvalue = child.getAttributeValue(attrname);
						map.put(lastname + "." + child.getName() + "."
								+ attrname, attrvalue);
						// map.put( attrname, attrvalue);
					}
				}
			}
		}
	}

2 一级一级的取元素的值

    public static Map<String, Object> xmlToMap(String xml, String encoding)
            throws UnsupportedEncodingException
    {
        byte[] xmlBytes = xml.getBytes(encoding);
        return xmlByteToMap(xmlBytes);
    }
    public static Map<String, Object> xmlByteToMap(byte[] xml)
    {
        Map<String, Object> map = new ListOrderedMap();
        if (new String(xml).trim().length() != 0)
        {
            SAXBuilder builder = new SAXBuilder();
            try
            {
                ByteArrayInputStream stringInputStream = new ByteArrayInputStream(
                        xml);
                Document doc = builder.build(stringInputStream);
                Element root = doc.getRootElement();
                elementToMap(root, map);
            }
            catch (JDOMException e)
            {
                logger.error(e.getMessage());
            }
            catch (IOException e)
            {
                logger.error(e.getMessage());
            }
        }
        return map;
    }
    private static Map<String, Object> elementToMap(Element element,
            Map<String, Object> map)
    {
        List children = element.getChildren();
        for (Iterator localIterator = children.iterator(); localIterator.hasNext();)
        {
            Object object = localIterator.next();
            
            Element ele = (Element)object;
            if (ele.getChildren().size() > 0)
            {
                Map hashMap = new ListOrderedMap();
                if (!map.containsKey(ele.getName()))
                {
                    map.put(ele.getName(), elementToMap(ele, hashMap));
                }
                else
                {
                    Object value = map.get(ele.getName());
                    if ((value instanceof List))
                    {
                        ((List)value).add(elementToMap(ele, hashMap));
                        map.put(ele.getName(), value);
                    }
                    else
                    {
                        List<Object> array = new ArrayList<Object>();
                        array.add((Map)value);
                        array.add(elementToMap(ele, hashMap));
                        map.put(ele.getName(), array);
                    }
                    
                }
                
            }
            else if (!map.containsKey(ele.getName()))
            {
                map.put(ele.getName(), ele.getTextTrim());
            }
            else
            {
                Object value = map.get(ele.getName());
                if ((value instanceof List))
                {
                    ((List)value).add(ele.getTextTrim());
                    map.put(ele.getName(), value);
                }
                else
                {
                    List<Object> array = new ArrayList<Object>();
                    array.add(value);
                    array.add(ele.getTextTrim());
                    map.put(ele.getName(), array);
                }
            }
            
        }
        
        return map;
    }
	
最后再来一个map的打印

	public static void printMap(Map<String, Object> map) {
		Iterator<String> keys = map.keySet().iterator();
		while (keys.hasNext()) {
			String key = keys.next();
			System.out.println(key + ":" + map.get(key));
		}
	}

Android中的xml解析
----------------------

	
写到这里，不禁又想起了android解析方式 sax和dom

SAX(Simple API for XML) 使用流式处理的方式，它并不记录所读内容的相关信息。它是一种以事件为驱动的XML API，解析速度快，占用内存少。使用回调函数来实现。 缺点是不能倒退。

DOM(Document Object Model) 是一种用于XML文档的对象模型，可用于直接访问XML文档的各个部分。它是一次性全部将内容加载在内存中，生成一个树状结构,它没有涉及回调和复杂的状态管理。 缺点是加载大文档时效率低下。

Pull内置于Android系统中。也是官方解析布局文件所使用的方式。Pull与SAX有点类似，都提供了类似的事件，如开始元素和结束元素。不同的是，SAX的事件驱动是回调相应方法，需要提供回调的方法，而后在SAX内部自动调用相应的方法。而Pull解析器并没有强制要求提供触发的方法。因为他触发的事件不是一个方法，而是一个数字。它使用方便，效率高。

SAX、DOM、Pull的比较:

内存占用：SAX、Pull比DOM要好；
编程方式：SAX采用事件驱动，在相应事件触发的时候，会调用用户编好的方法，也即每解析一类XML，就要编写一个新的适合该类XML的处理类。DOM是W3C的规范，Pull简洁。
访问与修改:SAX采用流式解析，DOM随机访问。
访问方式:SAX，Pull解析的方式是同步的，DOM逐字逐句。

官方解析xml配置居然用的Pull
举个例子吧

	public static List<Person> getPersons(InputStream inStream)
            throws Exception {
        List<Person> persons = new ArrayList<Person>();
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        DocumentBuilder builder = factory.newDocumentBuilder();
        Document document = builder.parse(inStream);
        Element root = document.getDocumentElement();
        NodeList personNodes = root.getElementsByTagName("person");
        for (int i = 0; i < personNodes.getLength(); i++) {
            Element personElement = (Element) personNodes.item(i);
            int id = new Integer(personElement.getAttribute("id"));
            Person person = new Person();
            person.setId(id);
            NodeList childNodes = personElement.getChildNodes();
            for (int y = 0; y < childNodes.getLength(); y++) {
                if (childNodes.item(y).getNodeType() == Node.ELEMENT_NODE) {
                    if ("name".equals(childNodes.item(y).getNodeName())) {
                        String name = childNodes.item(y).getFirstChild()
                                .getNodeValue();
                        person.setName(name);
                    }
                    else if ("age".equals(childNodes.item(y).getNodeName())) {
                        String age = childNodes.item(y).getFirstChild()
                                .getNodeValue();
                        person.setAge(new Short(age));
                    }
                }
            }
            persons.add(person);
        }
        inStream.close();
        return persons;
    }

