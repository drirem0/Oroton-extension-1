/*
 ***************************************************************
 *                                                             *
 *                           NOTICE                            *
 *                                                             *
 *   THIS SOFTWARE IS THE PROPERTY OF AND CONTAINS             *
 *   CONFIDENTIAL INFORMATION OF INFOR AND/OR ITS AFFILIATES   *
 *   OR SUBSIDIARIES AND SHALL NOT BE DISCLOSED WITHOUT PRIOR  *
 *   WRITTEN PERMISSION. LICENSED CUSTOMERS MAY COPY AND       *
 *   ADAPT THIS SOFTWARE FOR THEIR OWN USE IN ACCORDANCE WITH  *
 *   THE TERMS OF THEIR SOFTWARE LICENSE AGREEMENT.            *
 *   ALL OTHER RIGHTS RESERVED.                                *
 *                                                             *
 *   (c) COPYRIGHT 2020 INFOR.  ALL RIGHTS RESERVED.           *
 *   THE WORD AND DESIGN MARKS SET FORTH HEREIN ARE            *
 *   TRADEMARKS AND/OR REGISTERED TRADEMARKS OF INFOR          *
 *   AND/OR ITS AFFILIATES AND SUBSIDIARIES. ALL RIGHTS        *
 *   RESERVED.  ALL OTHER TRADEMARKS LISTED HEREIN ARE         *
 *   THE PROPERTY OF THEIR RESPECTIVE OWNERS.                  *
 *                                                             *
 ***************************************************************
 */

 import groovy.lang.Closure
 import java.time.LocalDate;
 import java.time.LocalDateTime;
 import java.time.format.DateTimeFormatter;
 import groovy.json.JsonSlurper;
 import java.math.BigDecimal;
 import java.math.RoundingMode;
 import java.text.DecimalFormat;
 import java.util.regex.Matcher;
 import java.util.regex.Pattern;

/*
 *Modification area - M3
 *Nbr               Date      User id     Description
 *AOS_R_200         20220824  RDRIESSEN   Mods BF0200- Display PO Related Data with GLS210 GL accounting string as input parameters
 *
*/

public class List extends ExtendM3Transaction {
  private final MIAPI mi;
  private final DatabaseAPI database;
  private final MICallerAPI miCaller;
  private final LoggerAPI logger;
  private final ProgramAPI program;
  private final IonAPI ion;
  
  //Input fields
  private String puno;
  private String cono;
  private String divi;
  private String ait1;
  private String ait2;
  private String ait3;
  private String ait4;
  private String ait5;
  private String ait6;
  private String ait7;
  private String acdt;
  private String pnli;
  private String ttyp;
  private String vser;
  private String vono;
  private String acam;
  private String yea4;
  private String anbr; 
  private String cucd;  
  private int XXCONO;
  private String XXDIVI;  
  private double sum;
  private boolean found;
  
 /*
  * Get Purchase Authorisation extension table row
 */
  public List(MIAPI mi, DatabaseAPI database, MICallerAPI miCaller, LoggerAPI logger, ProgramAPI program, IonAPI ion) {
    this.mi = mi;
    this.database = database;
  	this.miCaller = miCaller;
  	this.logger = logger;
  	this.program = program;
	  this.ion = ion;
    
  }
  
  public void main() {
    
    cono = mi.inData.get("CONO") == null ? '' : mi.inData.get("CONO").trim();
    if (cono == "?") {
      cono = "";
    }
    
    divi = mi.inData.get("DIVI") == null ? '' : mi.inData.get("DIVI").trim();
    if (divi == "?") {
      divi = "";
    }
    
    ait1 = mi.inData.get("AIT1") == null ? '' : mi.inData.get("AIT1").trim();
    if (ait1 == "?") {
      ait1 = "";
    }
    
    ait2 = mi.inData.get("AIT2") == null ? '' : mi.inData.get("AIT2").trim();
    if (ait2 == "?") {
      ait2 = "";
    }	
  	
    ait3 = mi.inData.get("AIT3") == null ? '' : mi.inData.get("AIT3").trim();
    if (ait3 == "?") {
      ait3 = "";
    }
    
    ait4 = mi.inData.get("AIT4") == null ? '' : mi.inData.get("AIT4").trim();
    if (ait4 == "?") {
      ait4 = "";
    }
    
  	ait5 = mi.inData.get("AIT5") == null ? '' : mi.inData.get("AIT5").trim();
    if (ait5 == "?") {
      ait5 = "";
    }
    
    ait6 = mi.inData.get("AIT6") == null ? '' : mi.inData.get("AIT6").trim();
    if (ait6 == "?") {
      ait6 = "";
    }	
  	
    ait7 = mi.inData.get("AIT7") == null ? '' : mi.inData.get("AIT7").trim();
    if (ait7 == "?") {
      ait7 = "";
    }
    
    acdt = mi.inData.get("ACDT") == null ? '' : mi.inData.get("ACDT").trim();
    if (acdt == "?") {
      acdt = "";
    } 
  	
  	XXCONO = (Integer)program.LDAZD.CONO;
  	XXDIVI = (String)program.LDAZD.DIVI;
  	
  	
  	//Validate input fields
    if (!validateInput()) {
      return;
    }
    
    writePOEXT001(cono, divi, ait1, ait2, ait3, ait4, ait5, ait6, ait7, acdt);
    
  }
  
  boolean validateInput() { 
    if (!cono.isEmpty()) {
	    if (cono.isInteger()){
		    XXCONO= cono.toInteger();
			  } else {
				  mi.error("Company " + cono + " is invalid.");
				  return false;
			  }
		  } else {
			XXCONO= program.LDAZD.CONO;
	  }
	  
	  
	  if (!ait1.isEmpty()) { 
  	  found = false;
      DBAction queryFCHACC01 = database.table("FCHACC").index("20").selection("EAAITM").build();
      DBContainer FCHACC01 = queryFCHACC01.getContainer();
      FCHACC01.set("EACONO", XXCONO);
      FCHACC01.set("EAAITP", 1);
      FCHACC01.set("EAAITM", ait1);
      queryFCHACC01.readAll(FCHACC01, 3, 1, lstFCHACC);
      if (!found) {
        mi.error("Acc dimension 1 is invalid.");
        return;
      }
    }
    
    if (acdt.toInteger() != 0 && !isDateValid(acdt)) {
      mi.error("Transaction date is invalid.");
      return false;
    }
    return true;
  }
  
  
  /*
	 * isDateValid1 - validate datestring
	 *
	*/
  
  def isDateValid(String dateStr) {
    boolean dateIsValid = true;
    Matcher matcher=
      Pattern.compile("^((2000|2400|2800|(19|2[0-9](0[48]|[2468][048]|[13579][26])))0229)\$" 
        + "|^(((19|2[0-9])[0-9]{2})02(0[1-9]|1[0-9]|2[0-8]))\$"
        + "|^(((19|2[0-9])[0-9]{2})(0[13578]|10|12)(0[1-9]|[12][0-9]|3[01]))\$" 
        + "|^(((19|2[0-9])[0-9]{2})(0[469]|11)(0[1-9]|[12][0-9]|30))\$").matcher(dateStr);
    dateIsValid = matcher.matches();
    if (dateIsValid) {
      dateIsValid = LocalDate.parse(dateStr, DateTimeFormatter.ofPattern("yyyyMMdd")) != null;
    } 
    return dateIsValid;
  }
    
  
  /*
	 * writePOEXT001 - write PO Extension Records
	 *
	*/
  def writePOEXT001(String cono, String divi, String ait1, String ait2, String ait3, String ait4, String ait5, String ait6, String ait7, String acdt) {
  
    DBAction query = database.table("FGLEDG").index("10").selection("EGCONO", "EGDIVI", "EGYEA4", "EGAIT1", "EGAIT2", "EGAIT3", "EGAIT4", "EGAIT5", "EGAIT6", "EGAIT7", "EGACDT", "EGVSER", "EGVONO").build();
    DBContainer container = query.getContainer();
    container.set("EGCONO", cono.toInteger());
    container.set("EGDIVI", divi);
    container.set("EGAIT1", ait1);
    container.set("EGAIT2", ait2);
    container.set("EGAIT3", ait3);
    container.set("EGAIT4", ait4);
    container.set("EGAIT5", ait5);
    container.set("EGAIT6", ait6);
    container.set("EGAIT7", ait7);
    container.set("EGACDT", acdt.toInteger());
    query.readAll(container, 10, releasedItemProcessor);
   
  }
  
  /*
  * releasedItemProcessor - Callback function
  *
  */
   
  Closure<?> releasedItemProcessor = { DBContainer container ->
  
    cono = container.get("EGCONO");
    divi = container.get("EGDIVI");
    yea4 = container.get("EGYEA4");
    vser = container.get("EGVSER");
    vono = container.get("EGVONO");
    ait1 = container.get("EGAIT1");
    ait2 = container.get("EGAIT2");
    ait3 = container.get("EGAIT3");
    ait4 = container.get("EGAIT4");
    ait5 = container.get("EGAIT5");
    ait6 = container.get("EGAIT6");
    ait7 = container.get("EGAIT7");

    DBAction query1 = database.table("CINACC").index("21").selection("EZCONO", "EZDIVI", "EZYEA4", "EZAIT1", "EZAIT2", "EZAIT3", "EZAIT4", "EZAIT5", "EZAIT6", "EZAIT7", "EZVSER", "EZVONO", "EZVSER").build();
    DBContainer container1 = query1.getContainer();
    container1.set("EZCONO", cono.toInteger());
    container1.set("EZDIVI", divi);
    container1.set("EZYEA4", yea4.toInteger());
    container1.set("EZVSER", vser);
    container1.set("EZVONO", vono.toInteger());
    container1.set("EZAIT1", ait1);
    container1.set("EZAIT2", ait2);
    container1.set("EZAIT3", ait3);
    container1.set("EZAIT4", ait4);
    container1.set("EZAIT5", ait5);
    container1.set("EZAIT6", ait6);
    container1.set("EZAIT7", ait7);
    query1.readAll(container1, 11, releasedItemProcessor1);
  }
  
  
 /*
  * releasedItemProcessor1 - Callback function
  *
  */
  
  Closure<?> releasedItemProcessor1 = { DBContainer container1 ->
  
    cono = container1.get("EZCONO");
    divi = container1.get("EZDIVI");
    yea4 = container1.get("EZYEA4");
    vser = container1.get("EZVSER");
    vono = container1.get("EZVONO");
    anbr = container1.get("EZANBR");
  
    DBAction query2 = database.table("MITTRA").index("80").selection("MTCONO", "MTWHLO", "MTANBR", "MTRIDN", "MTRIDL").build();
    DBContainer container2 = query2.getContainer();
    container2.set("MTCONO", cono.toInteger());
    container2.set("MTANBR", anbr.toInteger());
    query2.readAll(container2, 2, 100, releasedItemProcessor2);
  
  }
  
   /*
  * releasedItemProcessor7 - Callback function
  *
  */
  
  Closure<?> releasedItemProcessor7 = { DBContainer container7 ->
  mi.outData.put("SINO", container7.get("F5SINO").toString());
  
  }
 
  Closure<?> releasedItemProcessor2 = { DBContainer container2 ->
     
    pnli = container2.get("MTRIDL");
     
    DBAction query4 = database.table("MPLINE").index("00").selection("IBCONO", "IBPUNO", "IBSUNO", "IBPITD", "IBITNO", "IBORQA", "IBPITT", "IBLNAM", "IBWHLO").build();
    DBContainer container4 = query4.getContainer();
    container4.set("IBCONO", cono.toInteger());
    container4.set("IBPUNO", container2.get("MTRIDN").toString());
    container4.set("IBPNLI", pnli.toInteger());
    container4.set("IBPNLS", "0".toInteger());
    
    if(query4.read(container4)) {
  
      DBAction query5 = database.table("CIDMAS").index("00").selection("IDCONO", "IDSUNO", "IDSUNM").build();
      DBContainer container5 = query5.getContainer();
      container5.set("IDCONO", cono.toInteger());
      container5.set("IDSUNO", container4.get("IBSUNO").toString());
    
      if(query5.read(container5)) {
        mi.outData.put("SUNM", container5.get("IDSUNM").toString());
      }
  
      DBAction query6 = database.table("MPHEAD").index("00").selection("IACONO", "IACUCD", "IASUNO").build();
      DBContainer container6 = query6.getContainer();
      container6.set("IACONO", cono.toInteger());
      container6.set("IAPUNO", container4.get("IBPUNO").toString());
    
      if (query6.read(container6)) {
        mi.outData.put("CUCD", container6.get("IACUCD").toString());
        mi.outData.put("SUNO", container6.get("IASUNO").toString());
      }
      
      ExpressionFactory expression = database.getExpressionFactory("FGINLI");
      expression = expression.gt("F5IVNA", "0");    
      DBAction query7 = database.table("FGINLI").index("20").matching(expression).selection("F5SINO").build();
      DBContainer container7 = query7.getContainer();
      container7.set("F5CONO", cono.toInteger());
      container7.set("F5PUNO", container4.get("IBPUNO").toString());
      container7.set("F5PNLI", pnli.toInteger());
      query7.readAll(container7, 3, releasedItemProcessor7);

      mi.outData.put("CONO", container4.get("IBCONO").toString());
      mi.outData.put("WHLO", container4.get("IBWHLO").toString());
      mi.outData.put("PUNO", container4.get("IBPUNO").toString());
      mi.outData.put("DIVI", divi.toString());
      mi.outData.put("PNLI", container4.get("IBPNLI").toString());
      mi.outData.put("ITNO", container4.get("IBITNO").toString());
      mi.outData.put("PITD", container4.get("IBPITD").toString());
      mi.outData.put("PITT", container4.get("IBPITT").toString());
      mi.outData.put("ORQA", container4.get("IBORQA").toString());
      mi.outData.put("LNAM", container4.get("IBLNAM").toString());
      mi.outData.put("WHLO", container4.get("IBWHLO").toString());
 
      mi.write();
  
    }
  
  }
  
   /*
  * lstFCHACC - Callback function
  *
  */
  
  Closure<?> lstFCHACC = { DBContainer FCHACC ->
    found = true;
  }
  
  
  
}
