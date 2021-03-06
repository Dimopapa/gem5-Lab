# Αρχιτεκτονική Υπολογιστών 2ο εργαστήριο - Αναφορά
### Ομάδα 36
#### Παπασπανόπουλος Δημοσθένης 8729
#### Φρατζέσκος Παναγιώτης 8939

## Ερωτήματα 

### Εκτέλεση SPEC CPU2006 Benchmarks στον gem5
### 1.1
 Οι βασικές παρέμετροι για τον επεξεργαστή που εξομοιώνει ο gem5 όσον αφορά το υποσύστημα μνήμης είναι οι παρακάτω:
 (Τις πληροφορίες τις αντλούμε από το αρχείο config.ini)
 
<pre>L1 Instruction Size = 32768   (32kB)      
L1 Instruction Associativity = 2
L1 Data Size = 65536          (64kB)
L1 Data Associativity = 2
L2 Size = 2097152             (2MB)
L2 Associativity = 8
Cache Line Size = 64</pre>


### 1.2
 Στο παρακάτω διάγραμμα βλέπουμε τις ζητούμενες τιμές για το κάθε benchmark.
 (Χρησιμοποιήθηκε ένα script για να μπορέσουμε να ομαδοποιήσουμε και να καταγράψουμε τα δεδομένα από το αρχείο stats.txt)
 
 
![image](https://user-images.githubusercontent.com/43822588/70311619-aaae7e00-181a-11ea-9b8f-66b33eb5ec6f.png) 

Σύμφωνα με το διάγραμμα μπορούμε να βγάλουμε κάποια ενδιαφέροντα συμπεράσματα σχετικά με την cache μνήμη. Παρατηρούμε ότι τα benchmark libm και sjeng έχουν τους μεγαλύτερους χρόνους που σημαίνει ότι θα έχουν και μεγαλύτερο CPI. Αυτό συμβαίνει επειδή το L2 Cache Miss Rate είναι πάρα πολύ μεγάλο (99,99%). Άρα κάθε φορά που υπάρχει Miss penalty στην L2 cache ο χρόνος εκτέλεσης αυξάνεται κατά πολύ καθως όπως γνωρίζουμε η L2 cache είναι πολύ πιο αργή απο την L1.

### 1.3
 Τρέξαμε τα ίδια benchmark με την προσθήκη της παραμέτρου για το ρολόι στο 1 Ghz _--cpu-clock=1GHz._
 Βλέπουμε σύμφωνα με το παρακάτω διάγραμμα ότι η παράμετρος που αλλάζει, άρα και αυτή που αναφέρεται στο ρολόι του CPU είναι η cpu_clk_domain.clock. Συμπεραίνουμε οτι τα CPU parts βρίσκονται στο system.cpu_clk_domain ενώ τα system parts βρίσκονται στο system.clk_domain .
 
 ![image](https://user-images.githubusercontent.com/43822588/70313494-68873b80-181e-11ea-876e-94b046f36f6c.png)

Αν προσθέσουμε έναν ακόμα επεξεργαστή, προφανως με τα ίδια χαρακτηριστικά, εικάζουμε πως δεν θα υπήρχε διαφορά στον χρόνο εκτέλεσης. Θα υπήρχε διαφορά μόνον εάν υπήρχε παραλληλισμός, που στην προκειμένη δεν υπάρχει.


### Design Exploration – Βελτιστοποίηση απόδοσης
### 2.1
Ζητήθηκε να γίνει μελέτη για τους παρακάτω παράγοντες και πως αυτοί επηρεάζουν το CPI των Benchmarks.
- L1 instruction cache size
- L1 instruction cache associativity
- L1 data cache size
- L1 data cache associativity
- L2 cache size
- L2 cache associativity
- Μέγεθος cache line

*bzip* __CPI:1.679650__

Σε αυτό το benchmark το CPI είναι ήδη πολύ κοντά στο 1. Με βάση τα αποτελέσματα σπό το stats.txt όσον αφορά τα miss rates το μόνο που θα μπορούσε να βοηθήσει στην επιπλέον μείωση του είναι η αύξηση του L1 data associativity (--> 8) και η αύξηση του L1 data size (--> 128kB)

*hmmer* __CPI:1.187917__

Ομοίως και εδώ προσεγγίζεται πολύ κοντά στο 1. Για την βελτίωση του οι ιδανικές αλλαγές θα αφορούν, σύμφωνα με τα αποτελέσματα στα miss rates, τους παράγοντες cache line size (--> 128) και L1 data size (-->128kB)

*mcf* __CPI:1.299095__

Εδω έχουμε μεγάλο miss rate στο L1 instruction cache οπότε πιθανές αλλαγές που θα βελτίωναν την απόδοσή του είναι η αύξηση του cache line size ή του L1 instruction associativity.

*libm* __CPI:3.493415__ 

Σε αυτή την περίπτωση το  L2 Cache Miss Rate είναι τεράστιο (>99,99%) οπότε χρειάζεται βελτιώση με την αύξηση των L2 size, associativity και του Cache Line size.

*sjeng* __CPI:10.270554__

Συγκριτικά με το libm και σε αυτή την περίπτωση επειδή οι μετρήσεις είναι παρόμοιες, πιθανές αλλαγές είναι οι ίδιες με τις παραπάνω.

### 2.2
Σύμφωνα με τις παραπάνω μελέτες και μετά από πολλές δοκιμές προκύπτουν τα παρακάτω διαγράμματα τα οποία δείχνουν την επίδραση κάθε παράγοντα στην απόδοση κάθε benchmark.

![image](https://user-images.githubusercontent.com/43822588/70329089-39cf8c00-1843-11ea-8907-0df2b8f84aa0.png)



![image](https://user-images.githubusercontent.com/43822588/70329113-494ed500-1843-11ea-8b97-aac253d503d0.png)



![image](https://user-images.githubusercontent.com/43822588/70329173-6be0ee00-1843-11ea-942d-966dc84ea63a.png)



![image](https://user-images.githubusercontent.com/43822588/70330140-a3e93080-1845-11ea-8341-f0433f92869c.png)



![image](https://user-images.githubusercontent.com/43822588/70329231-929f2480-1843-11ea-93d1-303168953ae2.png)



![image](https://user-images.githubusercontent.com/43822588/70329249-9df25000-1843-11ea-98aa-f50c23804321.png)



![image](https://user-images.githubusercontent.com/43822588/70329351-de51ce00-1843-11ea-9946-9e9de50f308c.png)

 Συνολικά τα γραφήματα έχουν λογική και δείχνουν ακρίβως αυτή την επιρροή (ή μη) του κάθε παράγοντα για το εκάστοτε benchmark.

### 3. Κόστος απόδοσης και βελτιστοποίηση κόστους/απόδοσης

Σύμφωνα με τα αποτελέσματα και τις παρατηρήσεις στα προηγούμενα βήματα, οι παράμετροι που επηρεάζουν το κόστος απόδοσης άρα και αυτοί που θα τους λάβουμε υπόψιν ως παράγοντες στην συνάρτηση κόστους μας είναι οι παρακάτω:

* L1 Cache size (instruction & data)
* L2 Cache size
* L1 Cache associativity (instruction & data)
* L2 Cache associativity
* Cache line size

Παρακάτω θα συγκρίνουμε τις παραμέτρους αυτούς μεταξύ τους και θα προσεγγίσουμε την επιρροή τους στο κόστος.

#### L1 Cache size - L2 Cache size
Σύμφωνα με μια αναζήτηση για τον ρόλο τους, μπορούμε να βγάλουμε συμπέρασμα πως η L1 cache είναι x5 φορές περίπου γρηγορότερη από την L2 cache. Βέβαια όπως ξέρουμε η L1 cache χωρίζεται σε instruction και data όπου προσπελάσσονται παράλληλα. Έτσι λοιπόν μπορούμε προσεγγιστικά να θεωρήσουμε πως ισχύει L1_cache_size = 10* L2_cache_size

#### L1 Cache associativity -  L2 Cache associativity
Επειδή ως γωστόν η L1 cache βρίσκεται πιο κοντά στον επεξεργαστή από την L2 (ένα επίπεδο) άρα θεωρητικά μπορούμε να πούμε πως το κόστος για να έχουμε πρόσβαση στην L1 είναι πιο "ακριβό" από αυτό στην L2 άρα και βγάζουμε την σχέση 
L1_cache_associativity = 2* L2_cache_associativity

#### Cache line size
Γνωρίζουμε οτι η L1 cache έχει αποκλειστικά σχέση με την ταχύτητα ενώ η L2 cache έχει σχέση και με την χωρητικότητα, επομένως το κόστος για όσον αφορά την L2 για το cache line size είναι πιο ακριβό από αυτό που έχει να κάνει με την L2. Οπότε θεωρούμε ότι επηρεάζεται x2 η L1.

Συνολικά αν προσθέσουμε όλες τις παραπάνω σχέσεις και θεωρώντας ότι το associativity και το cache line size εξαρτώνται άμεσα από το cache size, βγάζουμε μια προσεγγιστική συνάρτηση κόστους που έχει την μορφή :

Cost = 10 * L1_data_cache_size * ( 2 * L1_cache_associativity + cache_line_size) + 10 * L1_instruction_cache_size * ( 2 * L1_cache_associativity + cache_line_size) + L2_data_cache_size * ( L1_cache_associativity + 2 * cache_line_size)


Resources:
https://en.wikipedia.org/wiki/CPU_cache
https://cirosantilli.com/linux-kernel-module-cheat/?fbclid=IwAR3h1ny5hRoVUIGP1vtOsUAzqXYd69sHBAopwuveHZtTSbd_MFp0B4Nmp8c#gem5-cpu-types
http://gem5.org/InOrder
http://www.m5sim.org/SimpleCPU#BaseSimpleCPU
http://www.gem5.org/docs/html/minor.html
