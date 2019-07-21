## MigrationMiner
<p align="justified">
<b>MigrationMiner</b> is an open source tool that provides the developer with easy-to-use and comprehensive way of extracting, from given list of input projects, existing migrations between two third-party libraries using program analysis based on Abstract Syntax Tree (AST) code representation. In a nutshell, MigrationMiner (i) detects, (ii) extracts, (iii) filters, and (iv) collects code changes related to any performed migration. For a given input project, MigrationMiner <b>detects</b> any migration undergone between two java libraries and returns the names and versions of both retired and new libraries. Thereafter, MigrationMiner <b>extracts</b> the specific code changes, from the client code, and which belong to the migration changes (it should at least have one removed method from the retired library, and one added method from the new library) from all other unrelated code changes within the commits. Next, MigrationMiner <b>filters</b> code changes to only keep fragments that contain migration traces i.e., a code fragment, generated by the <i>diff</i> utility, which contains the removed and added methods, respectively from the retired and the new library. Finally, MigrationMiner <b>collects</b> the library API documentation that is associated with every method in the client code. The output of MigrationMiner, for each detected migration between two libraries, is a set of migration traces, with their code context, and their corresponding documentation.
 </p>


### When you use this tool, please cite this paper.

<pre>
@inproceedings{alrbaye2019MigrationMiner,
  title={MigrationMiner: An Automated Detection Tool of Third-Party Java Library Migration at the Method Level},
  author={Hussein Alrubaye, Mohamed Wiem Mkaouer, Ali Ouni},
  booktitle={2019 IEEE International Conference on Software Maintenance and Evolution (ICSME)},
  year={2019},
  organization={IEEE}
}
</pre>

 
## This tool is not limited to library migration. You could use tool for:
- Migration/upgrade for Android Apps at the method and library Level.
- Migration/upgrade for Java Maven Projects at method and library Level.
- Track third-party libraries upgrade/migration for a project.
- Download third-party libraries used at given commit and extract source code + class/methods signature from jar file.
- Download third-party libraries documentation used at given commit and convert jar file to a relational database.



## Prerequisites

* Install java JKD from [here](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html).
* Install Eclipse IDE for Java Developers from [here](https://www.eclipse.org/downloads/packages/).
* Install  MYSQL Server from [here](https://dev.mysql.com/downloads/installer/).


## How to install and run the tool

#### To run the project on your local machine you can follow one of these two tutorials:

#### A- Using the tutorial video
* You could run the project by following this [video](https://youtu.be/sAlR1HNetXc).

#### B- Using the following steps
* First you need to build the dataset, by running the following script Database/MigrationMinerDBSQL.sql.
Open a terminal and run the following commands
```sql
 mysql -u root -p
 source ./MigrationMinerDBSQL.sql
```
After running the commands, the database should be created with all tables and views.

* Open eclipse IDE then go to File-> import-> Maven-> existing Maven Projects-> Select MigrationMiner directory.
* Set your local MYSQL username and password in this file "DatabaseLogin.java", which lives under MigrationMiner/src/main/java/com/project/settings/DatabaseLogin.java.
* Update MigrationMiner/data/gitRepositories.csv with the list of git repositories that you want to use as input (they will be searched for potential library migrations).
* Go to your github account under Settings > Developer Settings > Personal Access Tokens, add new token. Use token to set your GitHub token in this file "GithubLogin.java", which lives under MigrationMiner/src/main/java/com/project/settings/GithubLogin.java. Your token will be used so that Migration Miner can search a large number of GitHub projects without authentication issues.
* (Optional) We print alot of logs, to avoid console buffer overflow. In eclipse IDE go to   preferences-> console-> limit console buffer size to small number such as 10000.
* Run the Main.java file that lives under MigrationMiner/src/main/java/com/main/parse/Main.java.


## Tool output

##### A- Ouput as Relational Database
* After running Main.java, the database Tables will be filled with any migration infomation found. For each potential migration, the following information can be found in database, whose schema is as follows:
 
   * Repositories: Has the list of projects that scanned by the tool.
   * AppCommits: Has the list of projects' commits information (Commit Id, developer name, Commit text, and commit date).
   * ProjectLibraries: List of libraries that were added or removed at every commit.
   * MigrationRules:  List of migration Rules that were detected from the Dataset.
   * MigrationSegments: List Of migration Fragments that were extract from software migration.
   * LibraryDocumenation: Library documentation associated with every library version that has been involved in any migration.

##### B- Ouput as HTML
   There will be a generated HTML file named "MigrationMinnerOutput.html" that has the summary of all migrations detected, and for each migration, all its corresponding code fragments along with their Library documentation. An illutrative example of this file is in the following picture:
   
![main](https://repository-images.githubusercontent.com/185124992/bcd2f000-6f9d-11e9-9040-fbc3190eb01a)


##### C- Ouput as Objects
After running Main.java, You could read the output as objects by writing the following code. or run [TestClient.java](https://github.com/hussien89aa/MigrationMiner/blob/master/MigrationMiner/src/main/java/com/main/parse/TestClient.java). That could help you to integrate the tool with your code.

```java
 
//Return list of migrations between two pairs of libraries( added/removed)
LinkedList<MigrationRule> migrationRules= new MigrationRuleDB().getMigrationRulesWithoutVersion(1);

for (MigrationRule migrationRule : migrationRules) {
 System.out.println("== Migration Rule "+ migrationRule.FromLibrary +
      " <==> "+  migrationRule.ToLibrary +"==");

 /*
 *  For every migrations, retrieve list of collected of fragments for migration at method level.
 *  every fragment has N added methods M removed methods
 */
 ArrayList<Segment> segmentList = new MigrationSegmentsDB().getSegmentsObj(migrationRule.ID);

 for (Segment segment : segmentList) {

  segment.print();

  // Print all removed method signatures With Docs
  printMethodWithDocs( migrationRule.FromLibrary,segment.removedCode);  

  // Print all added method signatures With Docs
  printMethodWithDocs( migrationRule.ToLibrary,segment.addedCode);

 } // End fragment for every migration

}  // End library migration


/* 
* This method takes list of methods signatures with library that methods belong to.
* It will print the signatures and Docs for every method
*/
void printMethodWithDocs(String libraryName,ArrayList<String> listOfMethods ) {

 // For every add method print the Docs
 for(String methodSignature: listOfMethods){

  // Convert  method signatures as String to Object
  MethodObj methodFormObj= MethodObj.GenerateSignature(methodSignature);

  //retrieve Docs from the library for method has that name
  ArrayList<MethodDocs>  toLibrary = new LibraryDocumentationDB()
                                           .getDocs( libraryName,methodFormObj.methodName);

  //Map method signatures to docs
  MethodDocs methodFromDocs = MethodDocs.GetObjDocs(toLibrary, methodFormObj);

  if(methodFromDocs.methodObj== null) {
   System.err.println("Cannot find Docs for: "+ methodSignature);
   continue;
  }
  methodFromDocs.print();      
 }
}
```
 
## MigrationMiner has been used so far in the following papers:

* Alrubaye, H., & Mkaouer, M. W. (2018, October). [Automating the detection of third-party Java library migration at the function level](https://dl.acm.org/citation.cfm?id=3291299). In Proceedings of the 28th Annual International Conference on Computer Science and Software Engineering (pp. 60-71). IBM Corp.
* Alrubaye, H., Mkaouer, & M. W., Ali, O (2019).[ On the Use of Information Retrieval to Automate the Detection of Third-Party Java Library Migration At The Function Level](https://dl.acm.org/citation.cfm?id=3339129), 27th IEEE/ACM International Conference on Program Comprehension 2019.
* Alrubaye, H., & Wiem, M. (2019).[ Variability in Library Evolution](https://books.google.de/books?hl=en&lr=&id=uPSDDwAAQBAJ&oi=fnd&pg=PA295&dq=Variability+in+Library+Evolution+hussein&ots=zX79FyHrY8&sig=1-SwLFJLHP44UISBx2iQQpZZSLE#v=onepage&q=Variability%20in%20Library%20Evolution%20hussein&f=false). Software Engineering for Variability Intensive Systems: Foundations and Applications, 295.
* Alrubaye, Hussein, Mohamed Wiem Mkaouer, Igor Khokhlov, Leon Reznik, Ali Ouni, and Jason Mcgoff. [Learning to Recommend Third-Party Library Migration Opportunities at the API Level](https://people.rit.edu/hat6622/papers/1906.02882.pdf). arXiv preprint arXiv:1906.02882 (2019).
* Aljohani, A. (2019).[ An empirical study on discovering a new self-admitted technical debt type-API-debt](https://scholarworks.rit.edu/theses/10065/). Thesis. Rochester Institute of Technology. 
* Alrubaye, H., Mkaouer, & M. W., Ali, O (2019). [MigrationMiner: An Automated Detection Tool of Third-Party Java Library Migration at the Method Level](https://arxiv.org/pdf/1907.02997.pdf). 2019 IEEE International Conference on Software Maintenance and Evolution (ICSME).
* Alrubaye, Hussein, Deema AlShoaibi, Mohamed Wiem Mkaouer, Ali Ouni, . [How Does API Migration Impact Software Quality and Comprehension? An Empirical Study](https://people.rit.edu/hat6622/papers/API_Migration_Empirical.pdf). arXiv preprint arXiv:1906.02882 (2019). 
 
## License

This software is licensed under the [MIT license](https://opensource.org/licenses/MIT).
