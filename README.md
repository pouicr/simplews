Derivate from http://tomee.apache.org/examples-trunk/simple-webservice/README.html


## @WebService

The following is all that is required.  No external xml files are needed.  This class placed in a jar or war and deployed into a compliant Java EE server like TomEE is enough to have the Calculator class discovered and deployed and the webservice online.

    import javax.ejb.Stateless;
    import javax.jws.WebService;

    @Stateless
    @WebService(
            portName = "CalculatorPort",
            serviceName = "CalculatorService",
            targetNamespace = "http://superbiz.org/wsdl",
            endpointInterface = "org.superbiz.calculator.ws.CalculatorWs")
    public class Calculator implements CalculatorWs {

       @Inject
       Operator delegate;

       public int sum(int add1, int add2) {
           return delegate.sum(add1,add2);
       }

       public int multiply(int mul1, int mul2) {
           return delegate.multiply(mul1,mul2);
       }
    }

## Operator

    public class Operator {

        public int sum(int add1, int add2) {
            return add1 + add2;
        }

        public int multiply(int mul1, int mul2) {
            return mul1 * mul2;
        }
    }

## @WebService Endpoint Interface

Having an endpoint interface is not required, but it can make testing and using the web service from other Java clients far easier.

    import javax.jws.WebService;
    
    @WebService(targetNamespace = "http://superbiz.org/wsdl")
    public interface CalculatorWs {
    
        public int sum(int add1, int add2);
    
        public int multiply(int mul1, int mul2);
    }

## Calculator WSDL

The wsdl for our service is autmatically created for us and available at `http://127.0.0.1:4204/Calculator?wsdl`.  In TomEE or Tomcat this would be at `http://127.0.0.1:8080/simple-webservice/Calculator?wsdl`

    <?xml version="1.0" encoding="UTF-8"?>
    <wsdl:definitions xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/" name="CalculatorService"
                      targetNamespace="http://superbiz.org/wsdl"
                      xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
                      xmlns:tns="http://superbiz.org/wsdl" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
      <wsdl:types>
        <xsd:schema attributeFormDefault="unqualified" elementFormDefault="unqualified"
                    targetNamespace="http://superbiz.org/wsdl" xmlns:tns="http://superbiz.org/wsdl"
                    xmlns:xsd="http://www.w3.org/2001/XMLSchema">
          <xsd:element name="multiply" type="tns:multiply"/>
          <xsd:complexType name="multiply">
            <xsd:sequence>
              <xsd:element name="arg0" type="xsd:int"/>
              <xsd:element name="arg1" type="xsd:int"/>
            </xsd:sequence>
          </xsd:complexType>
          <xsd:element name="multiplyResponse" type="tns:multiplyResponse"/>
          <xsd:complexType name="multiplyResponse">
            <xsd:sequence>
              <xsd:element name="return" type="xsd:int"/>
            </xsd:sequence>
          </xsd:complexType>
          <xsd:element name="sum" type="tns:sum"/>
          <xsd:complexType name="sum">
            <xsd:sequence>
              <xsd:element name="arg0" type="xsd:int"/>
              <xsd:element name="arg1" type="xsd:int"/>
            </xsd:sequence>
          </xsd:complexType>
          <xsd:element name="sumResponse" type="tns:sumResponse"/>
          <xsd:complexType name="sumResponse">
            <xsd:sequence>
              <xsd:element name="return" type="xsd:int"/>
            </xsd:sequence>
          </xsd:complexType>
        </xsd:schema>
      </wsdl:types>
      <wsdl:message name="multiplyResponse">
        <wsdl:part element="tns:multiplyResponse" name="parameters"/>
      </wsdl:message>
      <wsdl:message name="sumResponse">
        <wsdl:part element="tns:sumResponse" name="parameters"/>
      </wsdl:message>
      <wsdl:message name="sum">
        <wsdl:part element="tns:sum" name="parameters"/>
      </wsdl:message>
      <wsdl:message name="multiply">
        <wsdl:part element="tns:multiply" name="parameters"/>
      </wsdl:message>
      <wsdl:portType name="CalculatorWs">
        <wsdl:operation name="multiply">
          <wsdl:input message="tns:multiply" name="multiply"/>
          <wsdl:output message="tns:multiplyResponse" name="multiplyResponse"/>
        </wsdl:operation>
        <wsdl:operation name="sum">
          <wsdl:input message="tns:sum" name="sum"/>
          <wsdl:output message="tns:sumResponse" name="sumResponse"/>
        </wsdl:operation>
      </wsdl:portType>
      <wsdl:binding name="CalculatorServiceSoapBinding" type="tns:CalculatorWs">
        <soap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http"/>
        <wsdl:operation name="multiply">
          <soap:operation soapAction="" style="document"/>
          <wsdl:input name="multiply">
            <soap:body use="literal"/>
          </wsdl:input>
          <wsdl:output name="multiplyResponse">
            <soap:body use="literal"/>
          </wsdl:output>
        </wsdl:operation>
        <wsdl:operation name="sum">
          <soap:operation soapAction="" style="document"/>
          <wsdl:input name="sum">
            <soap:body use="literal"/>
          </wsdl:input>
          <wsdl:output name="sumResponse">
            <soap:body use="literal"/>
          </wsdl:output>
        </wsdl:operation>
      </wsdl:binding>
      <wsdl:service name="CalculatorService">
        <wsdl:port binding="tns:CalculatorServiceSoapBinding" name="CalculatorPort">
          <soap:address location="http://127.0.0.1:4204/Calculator?wsdl"/>
        </wsdl:port>
      </wsdl:service>
    </wsdl:definitions>

## Accessing the @WebService with javax.xml.ws.Service

In our testcase we see how to create a client for our `Calculator` service via the `javax.xml.ws.Service` class and leveraging our `CalculatorWs` endpoint interface.

With this we can get an implementation of the interfacce generated dynamically for us that can be used to send compliant SOAP messages to our service.

    import org.junit.BeforeClass;
    import org.junit.Test;
    
    import javax.ejb.embeddable.EJBContainer;
    import javax.xml.namespace.QName;
    import javax.xml.ws.Service;
    import java.net.URL;
    import java.util.Properties;
    
    import static org.junit.Assert.assertEquals;
    import static org.junit.Assert.assertNotNull;
    
    public class CalculatorTest {
    
        @BeforeClass
        public static void setUp() throws Exception {
            Properties properties = new Properties();
            properties.setProperty("openejb.embedded.remotable", "true");
            //properties.setProperty("httpejbd.print", "true");
            //properties.setProperty("httpejbd.indent.xml", "true");
            EJBContainer.createEJBContainer(properties);
        }
    
        @Test
        public void test() throws Exception {
            Service calculatorService = Service.create(
                    new URL("http://127.0.0.1:4204/Calculator?wsdl"),
                    new QName("http://superbiz.org/wsdl", "CalculatorService"));
    
            assertNotNull(calculatorService);
    
            CalculatorWs calculator = calculatorService.getPort(CalculatorWs.class);
            assertEquals(10, calculator.sum(4, 6));
            assertEquals(12, calculator.multiply(3, 4));
        }
    }
