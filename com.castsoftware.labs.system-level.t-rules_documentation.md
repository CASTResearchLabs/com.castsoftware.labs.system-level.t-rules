# _com.castsoftware.labs.system-level.t-rules_ Description
This extension provides sample Transaction-level quality rules:
* Super-additivity per Technical Criterion
  * Avoid transactions with too many severe Efficiency - Memory, Network and Disk Space Management issues along the path
  * Avoid transactions with too many severe Efficiency - SQL and Data Handling Performance issues along the path
  * Avoid transactions with too many severe Programming Practices - Error and Exception Handling issues along the path
* Unbalanced Open-Close
  * Avoid transactions with the capability to open a resource but without the capability to close it

To support drill-down and investigation into their findings, it also provides
* an MS Excel spreadsheet
* an extraction toolkit to feed Neo4j graph database and allow Linkurious-assisted browsing \(cf. dedicated appendix [Appendix - Neo4J/Linkurious tooling](https://github.com/CASTResearchLabs/com.castsoftware.labs.system-level.t-rules/blob/master/com.castsoftware.labs.system-level.t-rules_appendix_neo4j-linkurious.md) \)

# Approach
Create Transaction-level – therefore system-level by nature – Quality Rules

## Super-additivity transaction-level rules
### how? 
By compounding issues of a specific Efficiency-related Technical Criterion within each transactions and setting concentration thresholds:
* more than X distinct locations (X now a parameter)
* more than Y distinct rules (Y now a parameter)

### why? 
They find situations:
* more likely to detect real defects
* more discriminating to support remediation actions (by identifying fewer but certainly more beneficial situations)

## Unbalanced open-close
### how?
By looking for transaction with unbalanced capabilities such as 
* the ability to open without the ability to close (meaning that either the resource is never closed, either it's "supposed to be" the role of another component to do it ...)
* the ability to update and rollback data without the ability to rollback all the different kind of data

As of this release, target JEE database resource open / close API:
* JDBC
  * open: java.sql.DriverManager.getConnection(String)
  * close: java.sql.Connection.close()
* JDBC
  * open: java.sql.Connection.createStatement()
  * close: java.sql.Statement.close()
* JDBC
  * open: java.sql.Connection.prepareStatement(...)
  * close: java.sql.PreparedStatement.close()
* JDBC
  * open: java.sql.Connection.prepareCall(...)
  * close: java.sql.CallableStatement.close()
* JDBC
  * open: java.sql.PreparedStatement.executeQuery()
  * close: java.sql.ResultSet.close()
* JPA
  * open: javax.persistence.Persistence.createEntityManagerFactory(String)
  * close: javax.persistence.EntityManagerFactory.close()
* JPA
  * open:  javax.persistence.EntityManagerFactory.createEntityManager()
  * close:  javax.persistence.EntityManager.close()
* Hibernate
  * open: org.hibernate.SessionFactory.openSession()
  * close: org.hibernate.Session.close()
* Hibernate
  * open: org.hibernate.cfg.Configuration.buildSessionFactory()
  * close: org.hibernate.SessionFactory.close()
* Spring
  * open: org.springframework.orm.hibernate3.SessionFactoryUtils.getSession(...)
  * close: org.springframework.orm.hibernate3.SessionFactoryUtils.closeSession(...)

### why? 
They find situations:
* more likely to detect real defects because capabilities should be balanced (either the transaction can open resource and it must be able to close it, either it can't and must rely one some other components to open and to close it) 

# In what situation should you install this extension?
If you are interested in getting a preview of what Transaction-assisted assessment could be.

# Release notes
## 1.0.0
Initial release delivering 2 new quality rules:
* Avoid transactions with too many severe Efficiency - Memory, Network and Disk Space Management issues along the path
* Avoid transactions with too many severe Efficiency - SQL and Data Handling Performance issues along the path

## 1.1.0
Release 1.1.0 adds to the delivery 1 new quality rule:
* Avoid transactions with too many severe Programming Practices - Error and Exception Handling issues along the path
The delivered rules are now fed by parameters to control the minimal number of distinct objects (respectively rules) to find in a single transaction to consider it a violation.

## 1.2.0
Release 1.2.0 adds to the delivery new quality rules: Avoid transactions with the capability to open a resource but without the capability to close it
It also adds to the delivery an MS Excel spreadsheet to drill-down into the results of the TC-level transaction rules.

## 1.3.0
Release 1.3.0 adds to the delivery an extraction toolkit to drill-down into the results of the transaction rules in Neo4j / Linkurious.
It also includes transaction graph extension to improve accuracy of all the transaction rules.

# _com.castsoftware.labs.system-level.t-rules_ technology support
All technologies configured to be analyzed by AIP are supported, although only JEE applications are supported for unbalanced open-close identification.

# Function Point, Quality and Sizing support
This extension is designed to extend Quality Assessments with system-level capabilities via new Quality Rules.

# CAST AIP compatibility

This extension is compatible with:
* 8.1.x out-of-the-box (AIP release used for the tests)
* 8.0.x after changing the nuspec file (there is no reason the extension would not work but it was not tested)

# Supported DBMS servers
This extension is currently supported on CAST databases installed on CAST Storage Server. All other supported RDBMS are not supported.


# Prerequisites
* An installation of any compatible release of CAST AIP (see table above)


# Bug Fix List
N/A

# Download and installation instructions

Please see:  [Extension Link Installation](http://doc.castsoftware.com/display/DOCEXT/Extension+download+and+installation)

The installation steps are the following:
* download the extension through the CAST Extension Downloader using the https://extend.castsoftware.com:443/labs download server
* open Server Manager 8.1+
* select the existing set of databases to update / install a new set of databases
* manage extensions of the existing set of database / follow the installation wizard up to the manage extension pane
* select _com.castsoftware.labs.system-level.t-rules.1.3.0_
* run the update / the installation  
* open CAST Management Studio
* import the Assessment Model from the Dashboard Service processed in steps #3 to #6 above; this is a *Mandatory* step to start computing the new indicator, one MUST import and use the Assessment Model from the Dashboard that was updated with the extension

# Packaging, delivering and analyzing your source code
Packaging, delivering and analyzing your source code is performed the same way as usual.

To get results:
* run a new snapshot

# What results can you expect?

## Objects
N/A

## Links
N/A

## Quality rules
List of new Quality Rules:
* Avoid transactions with too many severe Efficiency - SQL and Data Handling Performance issues along the path
* Avoid transactions with too many severe Efficiency - Memory, Network and Disk Space Management issues along the path
* Avoid transactions with the capability to open a resource but without the capability to close it
* Avoid transactions with too many severe Programming Practices - Error and Exception Handling issues along the path

## Technical Criteria
New quality rules results contributes to the following Technical Criteria:
* Efficiency - SQL and data handling performance
  * Avoid transactions with too many severe Efficiency - SQL and Data Handling Performance issues along the path
* Efficiency - Memory, network, and disk space management
  * Avoid transactions with too many severe Efficiency - Memory, Network and Disk Space Management issues along the path
  * Avoid transactions with the capability to open a resource but without the capability to close it
* Programming Practices - Error and Exception Handling
  * Avoid transactions with too many severe Programming Practices - Error and Exception Handling issues along the path

Therefore, you can expect grade variations.

## Business Criteria
New quality rules results contributes to the following Technical Criteria:
* Efficiency - SQL and data handling performance
* Efficiency - Memory, network, and disk space management
* Programming Practices - Error and Exception Handling

Therefore, you can expect grade variations for their parents business criteria.

# Limitations
N/A
