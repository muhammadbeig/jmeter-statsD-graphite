# jmeter-statsD-graphite
This JSR223 listener for Jmeter pushes jmeter sampler response times to graphite


Create a JSR223 listener in jmeter with the following code and place it at a level to listen from all samplers whose response time is required to be pushed to graphite.
The following code goes to the JSR223 listener:

import java.io.IOException;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.Socket;
import java.net.UnknownHostException;

GRAPHITE_HOST = "127.0.0.1";
CARBON_PORT = 2003;
METRICS_PATH="tests.performance.jmeter.yourprojectorproductname.responsetime."+sampleResult.toString();

                try {
                        Socket socket = new Socket(GRAPHITE_HOST,CARBON_PORT);
                        OutputStream s = socket.getOutputStream();
                        PrintWriter pr = new PrintWriter(s, true);
                        log.info(METRICS_PATH+" "+sampleResult.getTime().toString()+" "+System.currentTimeMillis()/1000+"\n");
        
                        pr.print(METRICS_PATH+" "+sampleResult.getTime().toString()+" "+System.currentTimeMillis()/1000+"\n");

                        pr.close();
                        socket.close();
                } catch (UnknownHostException e) {
                        throw new GraphiteException("Unknown host: " + graphiteHost);
                        log.info("Exception occurred");
                } catch (IOException e) {
                        throw new GraphiteException("Error while writing data to graphite: " + e.getMessage(), e);
                }

The following piece of code (in a JSR223 listener in JMeter) sends metrics (response time statsD "timer" and error code statsD "counter" to graphite through statsD. It uses the UDP port that statsD listens in and hence is more scalable.


GRAPHITEHOST="${graphitehost}";
PREFIX="${graphiteprefix}"+sampleResult.toString();

		try {


			host=InetAddress.getByName("${graphitehost}");
			String mystr=PREFIX+":"+sampleResult.getTime().toString()+"|ms\r\n"+PREFIX+".responsecode_"+sampleResult.getResponseCode()+":1|c";

			byte[] message = mystr.getBytes();
			DatagramPacket packet = new DatagramPacket(message, message.length,host, 8125);
			
			DatagramSocket dsocket = new DatagramSocket();
      		dsocket.send(packet);
      		dsocket.close();
		} catch (UnknownHostException e) {
			throw new GraphiteException("Unknown host: " + host);

		} catch (IOException e) {
			throw new GraphiteException("Error while writing data to statsD socket: " + e.getMessage(), e);
		}
