<?php
/* HAPPY DENT – Dental Clinic CRM (single file demo)
   Laragon: put in www/happydent/index.php -> open http://localhost/happydent
   DB + tables + demo data are auto-created on first run (SQL is inside install()).
   Hostinger/InfinityFree: change DB_H/DB_U/DB_P/DB_N below.
   Demo login: admin@happydent.com / admin123  (Receptionist: ramesh@happydent.com / admin123) */
session_start();
const DB_H='localhost',DB_U='root',DB_P='',DB_N='happydent_crm';

function db(){static $c;if($c)return $c;
 $c=new PDO('mysql:host='.DB_H.';charset=utf8mb4',DB_U,DB_P,[PDO::ATTR_ERRMODE=>PDO::ERRMODE_EXCEPTION,PDO::ATTR_DEFAULT_FETCH_MODE=>PDO::FETCH_ASSOC]);
 try{$c->exec("CREATE DATABASE IF NOT EXISTS `".DB_N."` CHARACTER SET utf8mb4");}catch(Exception $e){}
 $c->exec("USE `".DB_N."`");
 if(!$c->query("SHOW TABLES LIKE 'staff'")->fetch())install($c);
 if(!$c->query("SHOW TABLES LIKE 'hospitals'")->fetch())migrate($c);
 if(!$c->query("SHOW TABLES LIKE 'holidays'")->fetch())$c->exec("CREATE TABLE holidays(id INT AUTO_INCREMENT PRIMARY KEY,hdate DATE,reason VARCHAR(100),doctor_id INT DEFAULT 0,UNIQUE KEY u(hdate,doctor_id))");
 return $c;}
function q($s,$a=[]){$st=db()->prepare($s);$st->execute($a);return $st;}
function h($s){return htmlspecialchars((string)$s,ENT_QUOTES,'UTF-8');}

function install($c){
 $sql=<<<'SQL'
CREATE TABLE patients(id INT AUTO_INCREMENT PRIMARY KEY,name VARCHAR(100) NOT NULL,phone VARCHAR(20),age INT,gender VARCHAR(10),email VARCHAR(100),address VARCHAR(255),history TEXT,status VARCHAR(10) DEFAULT 'Active',created TIMESTAMP DEFAULT CURRENT_TIMESTAMP);
CREATE TABLE doctors(id INT AUTO_INCREMENT PRIMARY KEY,name VARCHAR(100) NOT NULL,spec VARCHAR(100),phone VARCHAR(20),hours VARCHAR(50),status VARCHAR(10) DEFAULT 'Active');
CREATE TABLE staff(id INT AUTO_INCREMENT PRIMARY KEY,name VARCHAR(100) NOT NULL,role VARCHAR(20) DEFAULT 'Staff',email VARCHAR(100) UNIQUE,phone VARCHAR(20),password VARCHAR(255),status VARCHAR(10) DEFAULT 'Active');
CREATE TABLE appointments(id INT AUTO_INCREMENT PRIMARY KEY,patient_id INT,doctor_id INT,adate DATE,atime TIME,reason VARCHAR(150),status VARCHAR(12) DEFAULT 'Booked');
CREATE TABLE invoices(id INT AUTO_INCREMENT PRIMARY KEY,patient_id INT,amount DECIMAL(10,2),idate DATE,status VARCHAR(10) DEFAULT 'Pending');
CREATE TABLE attendance(id INT AUTO_INCREMENT PRIMARY KEY,staff_id INT,adate DATE,check_in TIME NULL,check_out TIME NULL);
CREATE TABLE whatsapp_logs(id INT AUTO_INCREMENT PRIMARY KEY,patient_id INT,phone VARCHAR(20),msg TEXT,created TIMESTAMP DEFAULT CURRENT_TIMESTAMP)
SQL;
 foreach(explode(';',$sql) as $s)$c->exec($s);
 $pw=password_hash('admin123',PASSWORD_DEFAULT);
 foreach([['Kavya R',28,'Female','9876543210'],['Sathish K',35,'Male','9845678901'],['Meena S',24,'Female','9789012345'],['Rajesh M',42,'Male','9087654321'],['Anitha V',30,'Female','8765432109']] as $p)
  $c->prepare("INSERT INTO patients(name,age,gender,phone,email,address) VALUES(?,?,?,?,?,'Thanjavur, Tamil Nadu')")->execute([$p[0],$p[1],$p[2],$p[3],strtolower(explode(' ',$p[0])[0]).'@gmail.com']);
 foreach([['Dr. Arun','General Dentist','9876501234','9:00 AM - 5:00 PM'],['Dr. Priya','Orthodontist','9786123450','10:00 AM - 6:00 PM'],['Dr. Karthik','Oral Surgeon','9845612345','9:00 AM - 4:00 PM']] as $d)
  $c->prepare("INSERT INTO doctors(name,spec,phone,hours) VALUES(?,?,?,?)")->execute($d);
 foreach([['Admin','Admin','admin@happydent.com'],['Ramesh','Receptionist','ramesh@happydent.com'],['Divya','Staff','divya@happydent.com'],['Suresh','Staff','suresh@happydent.com'],['Dr. Arun','Doctor','arun@happydent.com']] as $s)
  $c->prepare("INSERT INTO staff(name,role,email,password) VALUES(?,?,?,?)")->execute([$s[0],$s[1],$s[2],$pw]);
 $st=['Visited','Visited','Booked','Unvisited','Postponed','Skipped','Booked'];$rs=['Tooth pain','Cleaning','Root canal','Braces check','Extraction','Scaling','Filling'];
 foreach($st as $i=>$s)$c->prepare("INSERT INTO appointments(patient_id,doctor_id,adate,atime,reason,status) VALUES(?,?,?,?,?,?)")->execute([$i%5+1,$i%3+1,date('Y-m-d',strtotime('+'.($i>4?$i-4:0).' day')),sprintf('%02d:00:00',9+$i),$rs[$i],$s]);
 foreach([[1,2500,'Paid'],[2,4800,'Paid'],[3,1200,'Pending'],[4,6500,'Paid'],[5,3200,'Pending']] as $i)
  $c->prepare("INSERT INTO invoices(patient_id,amount,idate,status) VALUES(?,?,CURDATE(),?)")->execute($i);
}

function migrate($c){
 $c->exec("CREATE TABLE hospitals(id INT AUTO_INCREMENT PRIMARY KEY,name VARCHAR(100),area VARCHAR(100),address VARCHAR(255),phone VARCHAR(20))");
 foreach(["ALTER TABLE doctors ADD hospital_id INT,ADD email VARCHAR(100),ADD password VARCHAR(255)","ALTER TABLE patients ADD password VARCHAR(255)","ALTER TABLE appointments ADD hospital_id INT","ALTER TABLE attendance ADD utype VARCHAR(8) DEFAULT 'staff'"] as $x)$c->exec($x);
 $pw=password_hash('admin123',PASSWORD_DEFAULT);
 $c->exec("INSERT INTO hospitals(name,area,address,phone) VALUES('Keerthana Hospital','Thanjavur - Location 1','Thanjavur, Tamil Nadu','9000000001'),('Happy Dent Hospital','Thanjavur - Location 2','Thanjavur, Tamil Nadu','9000000002')");
 $c->exec("UPDATE doctors SET hospital_id=IF(id<=2,2,1),email=CASE id WHEN 1 THEN 'arun@happydent.com' WHEN 2 THEN 'priya@happydent.com' ELSE 'karthik@keerthana.com' END");
 $c->exec("UPDATE appointments SET hospital_id=IF(doctor_id<=2,2,1)");$c->exec("DELETE FROM staff WHERE role='Doctor'");
 foreach(['doctors','patients'] as $t)$c->prepare("UPDATE $t SET password=?")->execute([$pw]);
}
/* ---------- entity config (drives list + form + save) ---------- */
$AS=['Pending','Booked','Visited','Unvisited','Postponed','Skipped','Cancelled'];$AC=['Active','Inactive'];
$E=[
'patients'=>['t'=>'Patient Management','q'=>"SELECT * FROM patients ORDER BY id DESC",
 'show'=>['id'=>'ID','name'=>'Name','phone'=>'Phone','age'=>'Age','gender'=>'Gender','created'=>'Since','status'=>'Status'],
 'f'=>['name'=>['Full Name','text'],'phone'=>['Phone','text'],'age'=>['Age','number'],'gender'=>['Gender','sel',['Male','Female','Other']],'email'=>['Email','email'],'address'=>['Address','text'],'history'=>['Medical History / Remarks','area'],'password'=>['Login Password (blank = keep / staff123)','password'],'status'=>['Status','sel',$AC]]],
'doctors'=>['t'=>'Doctor Management','q'=>"SELECT d.*,h.name hn FROM doctors d LEFT JOIN hospitals h ON h.id=d.hospital_id ORDER BY d.id DESC",
 'show'=>['name'=>'Name','spec'=>'Specialization','hn'=>'Hospital','phone'=>'Phone','hours'=>'Working Hours','status'=>'Status'],
 'f'=>['name'=>['Name','text'],'spec'=>['Specialization','text'],'phone'=>['Phone','text'],'hours'=>['Working Hours','text'],'hospital_id'=>['Hospital','sel','hospitals'],'email'=>['Email (login)','email'],'password'=>['Password (blank = keep / staff123)','password'],'status'=>['Status','sel',$AC]]],
'staff'=>['t'=>'Staff Management','q'=>"SELECT * FROM staff ORDER BY id DESC",
 'show'=>['name'=>'Name','role'=>'Role','email'=>'Email','phone'=>'Phone','status'=>'Status'],
 'f'=>['name'=>['Name','text'],'role'=>['Role','sel',['Admin','Receptionist','Staff']],'email'=>['Email (login)','email'],'phone'=>['Phone','text'],'password'=>['Password (blank = keep / staff123)','password'],'status'=>['Status','sel',$AC]]],
'appointments'=>['t'=>'Appointments','q'=>"SELECT a.*,p.name pn,d.name dn,h.name hn FROM appointments a LEFT JOIN patients p ON p.id=a.patient_id LEFT JOIN doctors d ON d.id=a.doctor_id LEFT JOIN hospitals h ON h.id=a.hospital_id {W} ORDER BY adate DESC,atime",
 'show'=>['adate'=>'Date','atime'=>'Time','pn'=>'Patient','dn'=>'Doctor','hn'=>'Hospital','reason'=>'Reason','status'=>'Status'],
 'f'=>['patient_id'=>['Patient','sel','patients'],'hospital_id'=>['Hospital','sel','hospitals'],'doctor_id'=>['Doctor','sel','doctors'],'adate'=>['Date','date'],'atime'=>['Time','time'],'reason'=>['Reason','text'],'status'=>['Status','sel',$AS]]],
'invoices'=>['t'=>'Billing & Invoices','q'=>"SELECT i.*,CONCAT('INV-',LPAD(i.id,3,'0')) inv,p.name pn FROM invoices i LEFT JOIN patients p ON p.id=i.patient_id {W} ORDER BY i.id DESC",
 'show'=>['inv'=>'Invoice No','pn'=>'Patient','idate'=>'Date','amount'=>'Amount','status'=>'Status'],
 'f'=>['patient_id'=>['Patient','sel','patients'],'amount'=>['Amount (₹)','number'],'idate'=>['Date','date'],'status'=>['Status','sel',['Paid','Pending']]]],
'hospitals'=>['t'=>'Hospitals','q'=>"SELECT * FROM hospitals ORDER BY id",'show'=>['name'=>'Hospital','area'=>'Location','address'=>'Address','phone'=>'Phone'],'f'=>['name'=>['Hospital Name','text'],'area'=>['Location / Area','text'],'address'=>['Address','text'],'phone'=>['Phone','text']]],
];
$MENU=['dashboard'=>['🏠','Dashboard'],'book'=>['➕','Book Appointment'],'appointments'=>['📅','Appointments'],'patients'=>['🧑','Patients'],'doctors'=>['🩺','Doctors'],'staff'=>['👥','Staff'],'hospitals'=>['🏥','Hospitals'],'calendar'=>['🗓️','Calendar'],'invoices'=>['🧾','Billing'],'reports'=>['📊','Reports'],'whatsapp'=>['💬','WhatsApp'],'attendance'=>['⏰','Attendance'],'settings'=>['⚙️','Settings']];
$ACC=['Admin'=>array_diff(array_keys($MENU),['book']),'Receptionist'=>['dashboard','appointments','patients','doctors','calendar','invoices','whatsapp','attendance','settings'],'Doctor'=>['dashboard','appointments','patients','calendar','attendance','settings'],'Staff'=>['dashboard','attendance','settings'],'Patient'=>['dashboard','book','appointments','invoices']];
$p=$_GET['p']??'dashboard';$u=$_SESSION['u']??null;
function can($p){global $ACC,$u;return $u&&in_array($p,$ACC[$u['role']]??[]);}
function waLog($aid){$r=q("SELECT a.*,p.name pn,p.phone,d.name dn,h.name hn FROM appointments a JOIN patients p ON p.id=a.patient_id LEFT JOIN doctors d ON d.id=a.doctor_id LEFT JOIN hospitals h ON h.id=a.hospital_id WHERE a.id=?",[$aid])->fetch();if(!$r)return;
 $head=$r['dn']?"Your dental appointment has been confirmed.":"We have received your appointment request. A doctor will confirm it shortly.";
 $m="Hello {$r['pn']},\n$head\nDate: ".date('d F Y',strtotime($r['adate']))."\nTime: ".date('h:i A',strtotime($r['atime']))."\nReason: {$r['reason']}\n".($r['dn']?"Doctor: {$r['dn']}\n":"")."Hospital: {$r['hn']}\n\nThank you for choosing Happy Dent. We look forward to seeing you! 😊";
 q("INSERT INTO whatsapp_logs(patient_id,phone,msg) VALUES(?,?,?)",[$r['patient_id'],$r['phone'],$m]);}
function sw(){$t=['brown'=>['#3b2416','#c98a4b','Brown Gold'],'ocean'=>['#0f3a5f','#1e90d6','Ocean Blue'],'emerald'=>['#0f4d3a','#22a06b','Emerald'],'rose'=>['#7a1f45','#e0568a','Rose'],'violet'=>['#3d2a6b','#8a5cf0','Violet'],'dark'=>['#0d0a08','#e0a25a','Dark']];$o='<div class=sw>';foreach($t as $k=>$c)$o.="<button type=button title='{$c[2]}' style='background:linear-gradient(135deg,{$c[0]} 50%,{$c[1]} 50%)' onclick=\"setTheme('$k')\"></button>";return $o.'</div>';}
function isHol($d,$doc){$r=q("SELECT reason FROM holidays WHERE hdate=? AND doctor_id IN(0,?) LIMIT 1",[$d,$doc])->fetch();return $r?($r['reason']?:'Holiday'):null;}
function cap($h,$dt){return (int)q("SELECT COUNT(*) FROM doctors d WHERE d.hospital_id=? AND d.status='Active' AND NOT EXISTS(SELECT 1 FROM holidays x WHERE x.hdate=? AND x.doctor_id=d.id)",[$h,$dt])->fetchColumn();}
function taken($h,$dt,$tm){return (int)q("SELECT COUNT(*) FROM appointments WHERE hospital_id=? AND adate=? AND atime=? AND status NOT IN('Cancelled','Skipped')",[$h,$dt,$tm.':00'])->fetchColumn();}
function badge($v){return "<span class='b b-".h($v)."'>".h($v)."</span>";}
function opts($o){return is_string($o)?array_column(q("SELECT id,name FROM $o ORDER BY name")->fetchAll(),'name','id'):array_combine($o,$o);}

/* ---------- actions ---------- */
if(isset($_GET['logout'])){session_destroy();header('Location: ?');exit;}
if($_SERVER['REQUEST_METHOD']==='POST'){
 $a=$_POST['action']??'';
 try{
  if($a==='login'){$t=['patient'=>'patients','doctor'=>'doctors'][$_POST['as']??'']??'staff';$s=q("SELECT * FROM $t WHERE email=? AND status='Active'",[trim($_POST['email'])])->fetch();
   if($s&&password_verify($_POST['password'],$s['password']??'')){if($t==='patients')$s['role']='Patient';if($t==='doctors')$s['role']='Doctor';$_SESSION['u']=$s;header('Location: ?');exit;}$_SESSION['f']='❌ Invalid email or password (or account not activated yet)';header('Location: ?');exit;}
  if($a==='register'){$as=$_POST['as']??'patient';$em=trim($_POST['email']);$nm=trim($_POST['name']);$ph=trim($_POST['phone']);$pwh=password_hash($_POST['password'],PASSWORD_DEFAULT);
   if($as==='doctor'){if(q("SELECT 1 FROM doctors WHERE email=?",[$em])->fetch())throw new Exception('Email already registered');
    q("INSERT INTO doctors(name,spec,phone,hospital_id,email,password,status) VALUES(?,?,?,?,?,?,'Inactive')",[$nm,trim($_POST['spec']),$ph,(int)$_POST['hospital_id'],$em,$pwh]);
    $_SESSION['f']='✅ Doctor registered! Admin activate pannina piragu login pannalam';header('Location: ?');exit;}
   if($as==='staff'){if(q("SELECT 1 FROM staff WHERE email=?",[$em])->fetch())throw new Exception('Email already registered');
    q("INSERT INTO staff(name,role,email,phone,password,status) VALUES(?,'Staff',?,?,?,'Inactive')",[$nm,$em,$ph,$pwh]);
    $_SESSION['f']='✅ Staff registered! Admin activate pannina piragu login pannalam';header('Location: ?');exit;}
   if(q("SELECT 1 FROM patients WHERE email=?",[$em])->fetch())throw new Exception('Email already registered');
   q("INSERT INTO patients(name,phone,age,gender,email,password) VALUES(?,?,?,?,?,?)",[$nm,$ph,(int)$_POST['age'],$_POST['gender'],$em,$pwh]);
   $s=q("SELECT * FROM patients WHERE id=?",[db()->lastInsertId()])->fetch();$s['role']='Patient';$_SESSION['u']=$s;$_SESSION['f']='🎉 Welcome to Happy Dent! Book your first appointment';header('Location: ?p=book');exit;}
  if(can($p)){
   if($u['role']==='Patient'&&!in_array($a,['book','cancel','pw']))$a='';
   if($a==='book'){$h=(int)$_POST['h'];$dt=$_POST['dt'];$tm=$_POST['tm']??'';
    if(!q("SELECT 1 FROM hospitals WHERE id=?",[$h])->fetch()||$dt<date('Y-m-d')||!preg_match('/^\d\d:\d\d$/',$tm))throw new Exception('Invalid booking');
    if($hx=isHol($dt,0))throw new Exception("Clinic closed on that date: $hx");
    if(taken($h,$dt,$tm)>=cap($h,$dt))throw new Exception('Sorry, that slot is full. Pick another');
    if(q("SELECT 1 FROM appointments WHERE patient_id=? AND adate=? AND atime=? AND status NOT IN('Cancelled','Skipped')",[$u['id'],$dt,$tm])->fetch())throw new Exception('You already have an appointment at that time');
    q("INSERT INTO appointments(patient_id,hospital_id,adate,atime,reason,status) VALUES(?,?,?,?,?,'Pending')",[$u['id'],$h,$dt,$tm,trim($_POST['reason'])]);waLog(db()->lastInsertId());$_SESSION['f']='🎉 Request sent! A doctor will confirm your appointment soon';$go='p=appointments';}
   elseif($a==='cancel'){q("UPDATE appointments SET status='Cancelled' WHERE id=? AND patient_id=? AND status IN('Booked','Pending')",[(int)$_POST['id'],$u['id']]);$_SESSION['f']='Appointment cancelled';}
   elseif($a==='accept'&&$u['role']==='Doctor'){$id=(int)$_POST['id'];
    $ap=q("SELECT * FROM appointments WHERE id=? AND status='Pending' AND doctor_id IS NULL AND hospital_id=?",[$id,(int)$u['hospital_id']])->fetch();
    if(!$ap)throw new Exception('Request is no longer available');
    if($hx=isHol($ap['adate'],(int)$u['id']))throw new Exception("You are on leave that day: $hx");
    if(q("SELECT 1 FROM appointments WHERE doctor_id=? AND adate=? AND atime=? AND status NOT IN('Cancelled','Skipped')",[$u['id'],$ap['adate'],$ap['atime']])->fetch())throw new Exception('You already have an appointment at that time');
    $n=q("UPDATE appointments SET doctor_id=?,status='Booked' WHERE id=? AND status='Pending' AND doctor_id IS NULL",[$u['id'],$id])->rowCount();
    if($n){waLog($id);$_SESSION['f']='✅ Appointment accepted';}else throw new Exception('Another doctor already accepted this');}
   elseif($a==='save'&&isset($E[$p])){$id=(int)$_POST['id'];$v=[];
    foreach($E[$p]['f'] as $k=>$f){$x=trim($_POST[$k]??'');
     if($k==='password'){if($x==='')$x=$id?'':'staff123';if($x==='')continue;$x=password_hash($x,PASSWORD_DEFAULT);}
     $v[$k]=$x===''?null:$x;}
    if($p==='appointments'&&!empty($v['doctor_id'])&&($v['status']??'')==='Pending')$v['status']='Booked';
    if($p==='appointments'&&!$id&&($hx=isHol($v['adate'],(int)$v['doctor_id'])))throw new Exception("Holiday: $hx");
    if($p==='appointments'&&!empty($v['doctor_id'])&&q("SELECT 1 FROM appointments WHERE doctor_id=? AND adate=? AND atime=? AND status NOT IN('Cancelled','Skipped') AND id<>?",[$v['doctor_id'],$v['adate'],$v['atime'],$id])->fetch())throw new Exception('Slot already booked for this doctor');
    if($id)q("UPDATE $p SET ".implode(',',array_map(fn($k)=>"$k=?",array_keys($v)))." WHERE id=?",[...array_values($v),$id]);
    else{q("INSERT INTO $p(".implode(',',array_keys($v)).") VALUES(".implode(',',array_fill(0,count($v),'?')).")",array_values($v));if($p==='appointments')waLog(db()->lastInsertId());}
    $_SESSION['f']='✅ Saved successfully';}
   elseif($a==='del'&&isset($E[$p])){if($p==='staff'&&$_POST['id']==$u['id'])throw new Exception('You cannot delete yourself');q("DELETE FROM $p WHERE id=?",[(int)$_POST['id']]);$_SESSION['f']='🗑️ Deleted';}
   elseif($a==='att'){$sid=(int)$u['id'];$ut=$u['role']==='Doctor'?'doctor':'staff';
    if($_POST['type']==='in'){if(!q("SELECT 1 FROM attendance WHERE staff_id=? AND utype=? AND adate=CURDATE()",[$sid,$ut])->fetch())q("INSERT INTO attendance(staff_id,utype,adate,check_in) VALUES(?,?,CURDATE(),CURTIME())",[$sid,$ut]);}
    else q("UPDATE attendance SET check_out=CURTIME() WHERE staff_id=? AND utype=? AND adate=CURDATE()",[$sid,$ut]);$_SESSION['f']='✅ Attendance updated';}
   elseif($a==='wa'&&can('whatsapp')){$r=q("SELECT * FROM patients WHERE id=?",[(int)$_POST['pid']])->fetch();q("INSERT INTO whatsapp_logs(patient_id,phone,msg) VALUES(?,?,?)",[$r['id'],$r['phone'],trim($_POST['msg'])]);$_SESSION['f']='✅ Message logged – tap "Open WhatsApp" to send';}
   elseif($a==='hol'&&in_array($u['role'],['Admin','Receptionist','Doctor'])){$d=$_POST['d'];$o=$u['role']==='Doctor'?(int)$u['id']:0;
    if(!preg_match('/^\d{4}-\d\d-\d\d$/',$d))throw new Exception('Bad date');
    if(q("SELECT 1 FROM holidays WHERE hdate=? AND doctor_id=?",[$d,$o])->fetch()){q("DELETE FROM holidays WHERE hdate=? AND doctor_id=?",[$d,$o]);$_SESSION['f']='Holiday removed';}
    else{q("INSERT INTO holidays(hdate,reason,doctor_id) VALUES(?,?,?)",[$d,trim($_POST['reason'])?:'Holiday',$o]);$_SESSION['f']='🎉 Holiday added – booking closed for '.date('d M',strtotime($d));}}
   elseif($a==='pw'){q("UPDATE ".(['Patient'=>'patients','Doctor'=>'doctors'][$u['role']]??'staff')." SET password=? WHERE id=?",[password_hash($_POST['pw'],PASSWORD_DEFAULT),$u['id']]);$_SESSION['f']='✅ Password changed';}
   elseif($a==='st'&&can('appointments')){q("UPDATE appointments SET status=? WHERE id=?",[$_POST['status'],(int)$_POST['id']]);$_SESSION['f']='✅ Status updated';}
  }
 }catch(Exception $e){$_SESSION['f']='⚠️ '.(str_contains($e->getMessage(),'Duplicate')?'Email already exists':$e->getMessage());}
 header('Location: ?'.($go??$_SERVER['QUERY_STRING']));exit;
}
if($u&&$p==='reports'&&isset($_GET['csv'])){header('Content-Type: text/csv');header('Content-Disposition: attachment; filename=report.csv');$o=fopen('php://output','w');fputcsv($o,['Date','Patient','Doctor','Reason','Status']);
 foreach(q("SELECT a.adate,p.name,d.name dn,a.reason,a.status FROM appointments a JOIN patients p ON p.id=a.patient_id LEFT JOIN doctors d ON d.id=a.doctor_id WHERE adate BETWEEN ? AND ?",[$_GET['from'],$_GET['to']]) as $r)fputcsv($o,$r);exit;}
$flash=$_SESSION['f']??'';unset($_SESSION['f']);
$R=$u['role']??'';$df='';$dp=[];if($R==='Doctor'){$df=" AND (a.doctor_id=? OR (a.status='Pending' AND a.hospital_id=?))";$dp=[$u['id'],(int)$u['hospital_id']];}elseif($R==='Patient'){$df=' AND a.patient_id=?';$dp=[$u['id']];}
?><!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>Happy Dent – Dental Clinic CRM</title>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&family=Baloo+2:wght@500;700;800&family=Pacifico&display=swap" rel="stylesheet">
<script>try{document.documentElement.dataset.theme=localStorage.getItem('th')||'brown'}catch(e){}function setTheme(t){try{localStorage.setItem('th',t)}catch(e){}document.documentElement.dataset.theme=t}</script>
<style>
:root,[data-theme=brown]{--br:#3b2416;--br2:#5a3a24;--gd:#c98a4b;--cr:#f7efe4;--bg:#fbf6ee;--soft:#fbeedd;--soft2:#fdf9f3;--soft3:#f3e8d8;--line:#e3d6c4;--inp:#fffdfa;--card:#fff;--tx:#2a1a10;--mu:#6b5a4c;--hd:#3b2416;--btn:#3b2416;--btnt:#fff;--gr:#2f9e5b;--bl:#4a7fd4;--or:#e8a020;--pu:#8a5cc4;--rd:#d94a4a}
[data-theme=ocean]{--br:#0f3a5f;--br2:#1d5a8c;--gd:#1e90d6;--cr:#e6f2fb;--bg:#f1f7fc;--soft:#dcecf9;--soft2:#f4f9fd;--soft3:#e3eff9;--line:#c5dbec;--inp:#fbfdff;--card:#fff;--tx:#0e2233;--mu:#4a6378;--hd:#0f3a5f;--btn:#0f3a5f;--btnt:#fff}
[data-theme=emerald]{--br:#0f4d3a;--br2:#1b6f54;--gd:#22a06b;--cr:#e5f5ee;--bg:#f2faf6;--soft:#d9f0e5;--soft2:#f4fbf7;--soft3:#e2f3ea;--line:#c2e0d1;--inp:#fbfffd;--card:#fff;--tx:#0d2a20;--mu:#466456;--hd:#0f4d3a;--btn:#0f4d3a;--btnt:#fff}
[data-theme=rose]{--br:#7a1f45;--br2:#a12d5f;--gd:#e0568a;--cr:#fdeaf1;--bg:#fff5f9;--soft:#fbdce8;--soft2:#fff8fb;--soft3:#fce6ef;--line:#f0c8d8;--inp:#fffcfd;--card:#fff;--tx:#3a1024;--mu:#7a5060;--hd:#7a1f45;--btn:#7a1f45;--btnt:#fff}
[data-theme=violet]{--br:#3d2a6b;--br2:#5b3fa0;--gd:#8a5cf0;--cr:#efe9fd;--bg:#f7f4fe;--soft:#e6dcfb;--soft2:#faf8ff;--soft3:#ece5fc;--line:#d6c9f3;--inp:#fdfcff;--card:#fff;--tx:#21163d;--mu:#5c5078;--hd:#3d2a6b;--btn:#3d2a6b;--btnt:#fff}
[data-theme=dark]{--br:#0d0a08;--br2:#2a211a;--gd:#e0a25a;--cr:#241e19;--bg:#17130f;--soft:#3a2f25;--soft2:#2b241e;--soft3:#332a22;--line:#4a3f36;--inp:#1d1814;--card:#241e19;--tx:#f7eee2;--mu:#c2b3a2;--hd:#f7eee2;--btn:#e0a25a;--btnt:#1a120b}
*{box-sizing:border-box;margin:0;padding:0}body{font-family:'Poppins','Segoe UI',system-ui,sans-serif;font-weight:500;background:var(--bg);color:var(--tx);font-size:14px;transition:background .4s,color .4s}
a{color:inherit;text-decoration:none}button,input,select,textarea{font:inherit}
.side{position:fixed;inset:0 auto 0 0;width:220px;background:var(--br);color:#f6ecdf;padding:16px 10px;display:flex;flex-direction:column;z-index:50;transition:.25s;overflow-y:auto}
.logo{display:flex;align-items:center;gap:8px;font-weight:700;font-size:18px;color:#fff;padding:4px 8px 16px}.logo i{font-style:normal;font-size:24px}
.side a{display:flex;gap:10px;padding:10px 12px;border-radius:8px;margin-bottom:3px;color:#f6ecdf}.side a:hover{background:var(--br2)}.side a.on{background:var(--gd);color:#fff;font-weight:600}
.me{margin-top:auto;padding:12px 8px;border-top:1px solid #ffffff20;font-size:12px}.me b{display:block;color:#fff;font-size:13px}
.main{margin-left:220px;min-height:100vh}
.top{position:sticky;top:0;z-index:40;background:var(--card);display:flex;align-items:center;gap:12px;padding:12px 20px;box-shadow:0 1px 6px #0001}
.top h1{font-size:19px}.top .sp{flex:1}.burger{display:none;background:none;border:0;font-size:24px;cursor:pointer}
.search{padding:9px 14px;border:1px solid var(--line);border-radius:8px;background:var(--bg);min-width:200px;max-width:340px;width:100%}
.date{color:var(--mu);font-size:13px;white-space:nowrap}
.wrap{padding:20px}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:14px;margin-bottom:18px}
.stat{background:var(--card);border-radius:12px;padding:16px;box-shadow:0 2px 8px #0001;border-left:4px solid var(--gd)}.stat small{color:var(--mu)}.stat b{display:block;font-size:28px;margin-top:4px}
.grid2{display:grid;grid-template-columns:1fr 1.6fr;gap:18px}.card{background:var(--card);border-radius:12px;padding:18px;box-shadow:0 2px 8px #0001;margin-bottom:18px}.card h3{margin-bottom:12px;font-size:16px}
.tw{overflow-x:auto}table{width:100%;border-collapse:collapse;min-width:520px}th{text-align:left;color:var(--mu);font-size:12px;font-weight:600;padding:10px 8px;border-bottom:1px solid #eee}td{padding:10px 8px;border-bottom:1px solid var(--line);vertical-align:middle}tr:hover td{background:var(--soft2)}
.b{display:inline-block;padding:3px 10px;border-radius:6px;font-size:12px;font-weight:600;background:#eee}
.b-Visited,.b-Active,.b-Paid,.b-Present{background:#dff3e6;color:#1f7a44}.b-Booked{background:#dce8fb;color:#2b5aa8}.b-Unvisited,.b-Pending{background:#fcefd2;color:#9a6a0a}.b-Postponed{background:#eadffa;color:#6a3fa8}.b-Skipped{background:#fbdcdc;color:#a83232}.b-Cancelled,.b-Inactive{background:#e6e2dc;color:#666}
.btn{background:var(--btn);color:var(--btnt);border:0;padding:9px 16px;border-radius:8px;cursor:pointer;font-weight:600}.btn:hover{background:var(--gd)}.btn.g{background:var(--gr)}.btn.o{background:var(--card);color:var(--hd);border:1px solid var(--hd)}
.ic{background:none;border:0;cursor:pointer;font-size:16px;padding:3px}.act{white-space:nowrap}.act form{display:inline}
.bar{display:flex;gap:10px;margin-bottom:14px;flex-wrap:wrap;align-items:center}.bar .search{flex:1}
.donut{width:160px;height:160px;border-radius:50%;display:grid;place-items:center;margin:0 auto 14px}.donut div{width:104px;height:104px;background:var(--card);border-radius:50%;display:grid;place-items:center;text-align:center;font-size:12px;color:var(--mu)}.donut b{font-size:26px;color:var(--tx);display:block}
.lg{display:flex;flex-wrap:wrap;gap:6px 14px;justify-content:center;font-size:12px}.lg i{display:inline-block;width:10px;height:10px;border-radius:50%;margin-right:5px}
.flash{background:var(--br);color:#fff;padding:11px 18px;border-radius:8px;margin-bottom:14px;animation:fo 4s forwards}@keyframes fo{0%,80%{opacity:1}100%{opacity:0;height:0;padding:0;margin:0}}
.modal{position:fixed;inset:0;background:#0007;display:none;place-items:center;z-index:99;padding:14px}.modal.on{display:grid}
.mb{background:var(--card);border-radius:14px;padding:22px;width:100%;max-width:520px;max-height:92vh;overflow-y:auto}.mb h3{margin-bottom:14px}
.fld{margin-bottom:12px}.fld label{display:block;font-size:12px;color:var(--mu);margin-bottom:4px}.fld input,.fld select,.fld textarea{width:100%;padding:9px 11px;border:1px solid var(--line);border-radius:8px;background:var(--inp)}
.cal{display:grid;grid-template-columns:repeat(7,1fr);gap:4px}.cal .h{text-align:center;font-size:12px;color:var(--mu);padding:6px 0}.cal .d{min-height:74px;background:var(--soft2);border-radius:8px;padding:6px;cursor:pointer;border:1px solid transparent;font-size:12px}.cal .d:hover,.cal .d.sel{border-color:var(--gd)}.cal .d.today{background:var(--soft)}.dot{display:inline-block;width:8px;height:8px;border-radius:50%;margin:2px 2px 0 0}
.chart{display:flex;align-items:flex-end;gap:10px;height:190px;overflow-x:auto;padding-bottom:6px}.chart .c{flex:1;min-width:34px;display:flex;flex-direction:column;justify-content:flex-end;align-items:center;height:100%;font-size:10px;color:var(--mu)}.chart .c div{width:100%;max-width:26px}
.chat{background:var(--soft3);border-radius:12px;padding:14px;max-height:520px;overflow-y:auto}.msg{background:#dcf8c6;color:#1c2b1c;border-radius:10px;padding:10px 12px;margin-bottom:10px;max-width:520px;white-space:pre-line;font-size:13px}.msg small{display:block;color:var(--mu);margin-top:4px}
.login{min-height:100vh;display:grid;place-items:center;background:linear-gradient(135deg,var(--cr),#ead6bb);padding:16px}.login .card{width:100%;max-width:380px;text-align:center}
.login{position:relative;overflow:hidden;background:linear-gradient(120deg,var(--cr),var(--soft),var(--bg),var(--cr));background-size:300% 300%;animation:bgm 12s ease infinite;font-family:'Baloo 2','Segoe UI',sans-serif}
@keyframes bgm{0%,100%{background-position:0 50%}50%{background-position:100% 50%}}
.bgt span{position:absolute;bottom:-70px;opacity:.55;animation:rise linear infinite}
@keyframes rise{to{transform:translateY(-120vh) rotate(360deg)}}
.lc{position:relative;z-index:2;margin-top:70px;padding:8px 26px 26px;border-radius:26px;background:var(--card);box-shadow:0 20px 50px #3b241633;animation:pop .8s cubic-bezier(.2,1.4,.4,1) both}
@keyframes pop{from{opacity:0;transform:translateY(40px) scale(.9)}}
.mascot{width:150px;height:165px;margin:-70px auto 0;display:block;overflow:visible;animation:flo 3s ease-in-out infinite;filter:drop-shadow(0 10px 8px #3b241640)}
@keyframes flo{50%{transform:translateY(-12px) rotate(3deg)}}
.eye{transform-box:fill-box;transform-origin:center;animation:bl 4s infinite}
@keyframes bl{0%,94%,100%{transform:scaleY(1)}97%{transform:scaleY(.1)}}
.lc.shy .eye{animation:none;transform:scaleY(.08)}
.tw2{transform-box:fill-box;transform-origin:center;animation:tw 1.6s ease-in-out infinite}
@keyframes tw{50%{transform:scale(.3) rotate(90deg);opacity:.3}}
.brand{font-family:Pacifico,cursive;font-size:36px;color:var(--hd);font-weight:400;margin-top:4px}
.tag{font-weight:600;color:var(--gd);letter-spacing:.5px;margin-bottom:16px}
.lc .fld input{font-family:inherit;font-size:15px;padding:11px 14px;border-radius:12px;transition:.25s}
.lc .fld input:focus{outline:0;border-color:var(--gd);box-shadow:0 0 0 4px #c98a4b33;transform:translateY(-2px)}
.lc .btn{font-family:inherit;font-size:16px;padding:12px;border-radius:12px;background:linear-gradient(90deg,var(--br),var(--gd),var(--br));background-size:200%;animation:bgm 4s linear infinite;transition:.2s}
.lc .btn:hover{transform:scale(1.03)}
@media(prefers-reduced-motion:reduce){.login *,.login{animation:none!important}}
@media(max-width:1200px){.grid2{grid-template-columns:1fr}}
@media(max-width:900px){.side{transform:translateX(-100%)}.side.on{transform:none;box-shadow:0 0 0 100vw #0006}.main{margin-left:0}.burger{display:block}.date{display:none}}
@media(max-width:520px){.wrap{padding:12px}.top{padding:10px 12px}.top h1{font-size:16px}.stat b{font-size:22px}.cal .d{min-height:52px;padding:3px;font-size:11px}.dot{width:6px;height:6px}.btn{padding:8px 12px}}
@media(min-width:1600px){body{font-size:15px}.wrap{max-width:1600px;margin:auto}}
.tabs{display:flex;gap:6px;background:var(--soft3);padding:5px;border-radius:14px;margin-bottom:14px}.tabs button{flex:1;border:0;background:none;padding:9px 4px;border-radius:10px;cursor:pointer;font-family:inherit;font-weight:700;color:var(--hd);transition:.25s}.tabs button.on{background:var(--br);color:#fff;transform:scale(1.05);box-shadow:0 4px 10px #3b241640}
.card,.stat,.hero{animation:up .55s cubic-bezier(.2,.9,.3,1) both}@keyframes up{from{opacity:0;transform:translateY(24px)}}
.stat:nth-child(2){animation-delay:.08s}.stat:nth-child(3){animation-delay:.16s}.stat:nth-child(4){animation-delay:.24s}.stat:nth-child(5){animation-delay:.32s}.stat:nth-child(6){animation-delay:.4s}
.stat,.cal .d,.btn,.side a{transition:.25s}.stat:hover{transform:translateY(-6px);box-shadow:0 12px 24px #3b241626}.btn:hover{transform:translateY(-2px)}.side a:hover{transform:translateX(5px)}
.logo i{display:inline-block;animation:flo 3s ease-in-out infinite}.mb{animation:pop .4s cubic-bezier(.2,1.4,.4,1) both}.top h1{animation:up .5s both}
tr{animation:fi .5s both}@keyframes fi{from{opacity:0}}.b-Booked{animation:pl 2s infinite}@keyframes pl{50%{box-shadow:0 0 0 5px #4a7fd433}}
.donut{animation:sp 1.2s cubic-bezier(.2,.9,.3,1) both}@keyframes sp{from{transform:rotate(-180deg) scale(.6);opacity:0}}
.chart .c div{transform-origin:bottom;animation:gr 1s both}@keyframes gr{from{transform:scaleY(0)}}
.hero{background:linear-gradient(120deg,var(--br),var(--br2));color:#fff;border-radius:16px;padding:22px;display:flex;justify-content:space-between;align-items:center;gap:14px;flex-wrap:wrap;margin-bottom:18px}.hero h2{font-family:'Baloo 2',sans-serif}.hero .btn{background:var(--gd);animation:pl 2s infinite}
.hosp{display:grid;grid-template-columns:repeat(auto-fit,minmax(230px,1fr));gap:14px}.hc{display:block;padding:18px;border-radius:14px;border:2px solid var(--line);transition:.3s;background:var(--inp)}.hc:hover{transform:translateY(-5px);box-shadow:0 10px 20px #3b241622}.hc.on{border-color:var(--gd);background:var(--soft)}.hc span{font-size:30px;display:block;animation:flo 3s infinite}.hc small{display:block;color:var(--mu);margin-top:4px}
.chips{display:flex;gap:10px;flex-wrap:wrap}.chip{padding:10px 16px;border-radius:30px;border:2px solid var(--line);background:var(--card);transition:.2s}.chip small{display:block;font-size:11px;color:var(--mu)}.chip.on,.chip:hover{border-color:var(--gd);background:var(--soft)}
.slots{display:grid;grid-template-columns:repeat(auto-fill,minmax(96px,1fr));gap:10px}.slot{position:relative}.slot input{position:absolute;opacity:0;pointer-events:none}.slot span{display:block;text-align:center;padding:10px;border-radius:10px;border:2px solid var(--line);cursor:pointer;transition:.2s;background:var(--card)}.slot span:hover{transform:translateY(-3px);border-color:var(--gd)}.slot input:checked+span{background:var(--gd);color:#fff;border-color:var(--gd);transform:scale(1.07)}.slot.off span{opacity:.35;text-decoration:line-through;cursor:not-allowed;transform:none}
@media(prefers-reduced-motion:reduce){*{animation:none!important;transition:none!important}}
h1,h2,h3{font-family:'Baloo 2','Poppins',sans-serif}.card h3{color:var(--hd);font-size:18px}.stat b{font-family:'Baloo 2',sans-serif;color:var(--hd)}
input,select,textarea{color:var(--tx)}.brand{margin-bottom:12px}.lc .btn{color:#fff}th{color:var(--mu);font-weight:700}.side a{font-weight:600}.top h1{color:var(--hd)}
.sw{display:flex;gap:10px;flex-wrap:wrap}.sw button{width:38px;height:38px;border-radius:50%;border:3px solid var(--card);box-shadow:0 0 0 2px var(--line);cursor:pointer;transition:.2s}.sw button:hover{transform:scale(1.2) rotate(15deg)}
.lsw{position:fixed;top:14px;right:14px;z-index:5;background:var(--card);padding:8px 10px;border-radius:30px;box-shadow:0 4px 14px #0003}.lsw .sw{gap:6px}.lsw .sw button{width:24px;height:24px;border-width:2px}
.tpw{position:relative}.tp{display:none;position:absolute;right:0;top:42px;background:var(--card);padding:14px;border-radius:14px;box-shadow:0 10px 28px #0004;z-index:60;width:190px}.tp.on{display:block;animation:pop .3s both}
.cal .d.hol{background:#fbdcdc;color:#a83232;font-weight:600}.cal .d.lock{cursor:not-allowed;opacity:.85}.cal .d small{display:block;font-size:10px;line-height:1.2;margin-top:2px}
.att{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:16px}
.ac{background:var(--card);border-radius:18px;padding:22px 14px;text-align:center;box-shadow:0 2px 10px #0002;border-top:5px solid var(--line);animation:up .55s both;transition:.3s}.ac:hover{transform:translateY(-6px)}
.ac.in{border-color:#2f9e5b}.ac.done{border-color:#4a7fd4}.ac.out0{border-color:#e8a020}
.ob{width:88px;height:88px;margin:0 auto 12px;border-radius:50%;display:grid;place-items:center;font-size:42px;background:var(--soft);position:relative}.ob span{animation:flo 3s ease-in-out infinite}
.ob:after{content:'💤';position:absolute;right:-4px;bottom:-4px;font-size:18px;background:var(--card);border-radius:50%;width:32px;height:32px;display:grid;place-items:center;box-shadow:0 2px 6px #0004}
.ac.in .ob{animation:ring 1.6s infinite}.ac.in .ob:after{content:'🟢'}.ac.done .ob:after{content:'✅'}@keyframes ring{0%{box-shadow:0 0 0 0 #2f9e5b99}100%{box-shadow:0 0 0 20px #2f9e5b00}}
.ac small{display:block;color:var(--mu);margin-bottom:8px}.ac .tm{display:flex;flex-direction:column;gap:3px;font-size:12px;margin:10px 0 14px}.ac .tm i{font-style:normal}
.btn.ci{background:#2f9e5b;color:#fff}.btn.co{background:#e8742a;color:#fff}
.clock{font:800 34px 'Baloo 2',sans-serif;color:var(--hd);background:var(--soft);padding:6px 20px;border-radius:14px;letter-spacing:2px}
.bgt{position:absolute;inset:0;pointer-events:none}.tl{position:absolute;bottom:-280px;animation:drift linear infinite;will-change:transform}
@keyframes drift{from{transform:translateY(0) rotate(var(--a))}to{transform:translateY(-140vh) rotate(calc(var(--a) + 70deg))}}
.login:before{content:'';position:absolute;inset:0;background:radial-gradient(circle at 18% 22%,#ffffffaa,transparent 38%),radial-gradient(circle at 82% 80%,var(--soft),transparent 45%)}
/* ===== RESPONSIVE FIX: Mobile / Tablet / Laptop ===== */
html,body{max-width:100%;overflow-x:hidden}
.main,.wrap,.card,.grid2>*{min-width:0}
.modal{grid-template-columns:minmax(0,1fr);justify-items:center;overflow-y:auto}
.mb{min-width:0;width:100%}
.fld input,.fld select,.fld textarea{min-width:0;max-width:100%;display:block}
input[type=date],input[type=time]{min-width:0;max-width:100%}
img,svg{max-width:100%}
/* Laptop (1025px+) uses the default layout */
/* Tablet (601px - 1024px) */
@media(max-width:1024px){
 .grid2{grid-template-columns:1fr!important}
 .stats{grid-template-columns:repeat(auto-fit,minmax(140px,1fr))}
 .att{grid-template-columns:repeat(auto-fill,minmax(200px,1fr))}
 .mb{max-width:560px}
 .search{max-width:none;min-width:0}
}
/* Mobile (up to 600px) */
@media(max-width:600px){
 .top{flex-wrap:wrap;gap:8px;padding:10px 12px}
 .top .search{order:9;flex:1 1 100%;max-width:none;min-width:0}
 .top h1{font-size:16px}
 .wrap{padding:10px}
 .card{padding:14px}
 .stats{grid-template-columns:repeat(2,1fr);gap:10px}
 .stat{padding:12px}.stat b{font-size:22px}
 .hero{padding:16px}
 .hosp{grid-template-columns:1fr}
 .att{grid-template-columns:1fr}
 .slots{grid-template-columns:repeat(auto-fill,minmax(82px,1fr));gap:8px}
 .modal{padding:8px;align-items:start}
 .mb{padding:16px;border-radius:12px;max-height:96vh}
 .bar{gap:8px}
 .bar>*{flex:1 1 auto}
 .cal{gap:2px}.cal .d small{display:none}
 .tabs button{font-size:12px}
 .lc{padding:8px 16px 20px}
 .lsw{top:8px;right:8px}
 .chart .c{min-width:28px}
 .clock{font-size:26px;padding:4px 12px}
 .msg{max-width:100%}
}
</style></head><body>
<?php if(!$u): ?>
<link href="https://fonts.googleapis.com/css2?family=Baloo+2:wght@500;700;800&family=Pacifico&display=swap" rel="stylesheet">
<div class="login"><div class="bgt"><svg width="0" height="0" style="position:absolute" aria-hidden="true"><defs>
<linearGradient id="gSteel" x1="0" x2="1" y1="0" y2="0"><stop offset="0" stop-color="#5f6973"/><stop offset=".28" stop-color="#f7fafc"/><stop offset=".55" stop-color="#a5aeb8"/><stop offset=".8" stop-color="#eef2f5"/><stop offset="1" stop-color="#565f68"/></linearGradient>
<linearGradient id="gBlue" x1="0" x2="1" y1="0" y2="0"><stop offset="0" stop-color="#17569a"/><stop offset=".4" stop-color="#55bdf5"/><stop offset="1" stop-color="#17569a"/></linearGradient>
<linearGradient id="gWhite" x1="0" x2="1" y1="0" y2="0"><stop offset="0" stop-color="#c3ccd4"/><stop offset=".45" stop-color="#fff"/><stop offset="1" stop-color="#bcc6cf"/></linearGradient>
<linearGradient id="gGlass" x1="0" x2="1" y1="0" y2="0"><stop offset="0" stop-color="#bfe0f7" stop-opacity=".55"/><stop offset=".45" stop-color="#fff" stop-opacity=".9"/><stop offset="1" stop-color="#8fc0e8" stop-opacity=".55"/></linearGradient>
<linearGradient id="gFluid" x1="0" x2="1" y1="0" y2="0"><stop offset="0" stop-color="#e2c95a"/><stop offset=".5" stop-color="#fbf0a8"/><stop offset="1" stop-color="#d9bd48"/></linearGradient>
<radialGradient id="gMirr" cx=".35" cy=".3" r=".85"><stop offset="0" stop-color="#fff"/><stop offset=".4" stop-color="#c3e2f8"/><stop offset="1" stop-color="#4d82ad"/></radialGradient>
<linearGradient id="gBris" x1="0" x2="0" y1="0" y2="1"><stop offset="0" stop-color="#fff"/><stop offset="1" stop-color="#9fd0f2"/></linearGradient>
<symbol id="tMirror" viewBox="0 0 60 240"><circle cx="30" cy="32" r="25" fill="url(#gSteel)"/><circle cx="30" cy="32" r="20" fill="url(#gMirr)"/><ellipse cx="22" cy="23" rx="9" ry="5" fill="#fff" opacity=".65" transform="rotate(-35 22 23)"/><rect x="26.5" y="55" width="7" height="24" fill="url(#gSteel)"/><path d="M24 78h12l3 146q0 9-9 9t-9-9z" fill="url(#gSteel)"/><path d="M25 100h10M25 112h10M25 124h10M25 136h10M25 148h10" stroke="#3f4850" stroke-width="1.2" opacity=".55"/></symbol>
<symbol id="tProbe" viewBox="0 0 60 240"><path d="M30 118V34Q30 8 48 10" fill="none" stroke="url(#gSteel)" stroke-width="3.5" stroke-linecap="round"/><path d="M22 112h16l4 112q0 9-12 9t-12-9z" fill="url(#gSteel)"/><path d="M24 132h12M24 144h12M24 156h12M24 168h12M24 180h12" stroke="#3f4850" stroke-width="1.2" opacity=".55"/></symbol>
<symbol id="tBrush" viewBox="0 0 60 260"><path d="M21 76h18l4 40q1 8-2 16l-5 110q0 8-6 8t-6-8l-5-110q-3-8-2-16z" fill="url(#gBlue)"/><path d="M28 96h4v130h-4z" fill="#fff" opacity=".5"/><rect x="14" y="6" width="32" height="72" rx="9" fill="url(#gBlue)"/><path d="M17 11h5.5v9h-5.5zM24 11h5.5v9h-5.5zM31 11h5.5v9h-5.5zM38.5 11h5.5v9h-5.5zM17 23h5.5v9h-5.5zM24 23h5.5v9h-5.5zM31 23h5.5v9h-5.5zM38.5 23h5.5v9h-5.5zM17 35h5.5v9h-5.5zM24 35h5.5v9h-5.5zM31 35h5.5v9h-5.5zM38.5 35h5.5v9h-5.5zM17 47h5.5v9h-5.5zM24 47h5.5v9h-5.5zM31 47h5.5v9h-5.5zM38.5 47h5.5v9h-5.5zM17 59h5.5v9h-5.5zM24 59h5.5v9h-5.5zM31 59h5.5v9h-5.5zM38.5 59h5.5v9h-5.5z" fill="url(#gBris)"/></symbol>
<symbol id="tTube" viewBox="0 0 90 240"><rect x="30" y="6" width="30" height="30" rx="4" fill="url(#gBlue)"/><path d="M34 12v18M40 12v18M46 12v18M52 12v18" stroke="#0e3b6b" stroke-width="1" opacity=".4"/><rect x="34" y="34" width="22" height="12" fill="url(#gBlue)"/><path d="M14 46Q45 26 76 46L72 200H18z" fill="url(#gWhite)"/><path d="M15 92q15-16 30 0t30 0v40q-15 16-30 0t-30 0z" fill="url(#gBlue)" opacity=".92"/><rect x="18" y="200" width="54" height="30" fill="url(#gWhite)"/><path d="M18 205h54M18 211h54M18 217h54M18 223h54" stroke="#aab4bd" stroke-width="1.2"/></symbol>
<symbol id="tSyringe" viewBox="0 0 60 260"><rect x="29" y="0" width="2" height="54" fill="url(#gSteel)"/><path d="M25 54h10l3 14H22z" fill="url(#gSteel)"/><rect x="18" y="68" width="24" height="84" rx="5" fill="url(#gGlass)" stroke="#9cc3e0"/><rect x="20" y="92" width="20" height="56" fill="url(#gFluid)" opacity=".9"/><rect x="14" y="150" width="32" height="62" rx="7" fill="url(#gSteel)"/><rect x="3" y="158" width="54" height="9" rx="4.5" fill="url(#gSteel)"/><rect x="27" y="212" width="6" height="30" fill="url(#gSteel)"/><ellipse cx="30" cy="250" rx="15" ry="8" fill="none" stroke="url(#gSteel)" stroke-width="4"/></symbol>
<symbol id="tForceps" viewBox="0 0 100 240"><g fill="none" stroke-linecap="round"><path d="M38 232Q26 150 44 104Q52 70 46 18" stroke="url(#gSteel)" stroke-width="9"/><path d="M62 232Q74 150 56 104Q48 70 54 18" stroke="url(#gSteel)" stroke-width="9"/><path d="M36 220Q25 150 42 106" stroke="#fff" stroke-width="2" opacity=".5"/><path d="M58 108Q73 150 63 220" stroke="#fff" stroke-width="2" opacity=".4"/></g><circle cx="50" cy="96" r="5" fill="#7d8791"/></symbol>
</defs></svg><?php $T=[['Mirror',60,240],['Probe',60,240],['Brush',60,260],['Tube',90,240],['Syringe',60,260],['Forceps',100,240]];
for($i=0;$i<14;$i++){$t=$T[$i%6];$sc=mt_rand(45,100)/100;$w=round($t[1]*$sc*.9);$hh=round($t[2]*$sc*.9);$bl=$sc<.6?2.5:($sc<.75?1:0);
 echo"<svg class=tl width=$w height=$hh viewBox='0 0 {$t[1]} {$t[2]}' style='left:".mt_rand(0,94)."%;--a:".mt_rand(-45,45)."deg;animation-duration:".mt_rand(16,34)."s;animation-delay:-".mt_rand(0,30)."s;filter:drop-shadow(0 14px 10px #0004) blur({$bl}px)'><use href='#t{$t[0]}' width='{$t[1]}' height='{$t[2]}'/></svg>";}?></div>
<div class="lsw"><?=sw()?></div>
<div class="card lc" style="max-width:410px">
<svg class="mascot" viewBox="0 0 200 220"><path d="M100 30C70 10 25 20 25 70C25 105 45 120 50 155C54 190 62 205 76 205C92 205 90 165 100 165C110 165 108 205 124 205C138 205 146 190 150 155C155 120 175 105 175 70C175 20 130 10 100 30Z" fill="#fff" stroke="#c98a4b" stroke-width="6" stroke-linejoin="round"/>
<path d="M52 62Q58 42 80 36" stroke="#dbe8ff" stroke-width="9" fill="none" stroke-linecap="round"/>
<ellipse class="eye" cx="78" cy="88" rx="8" ry="11" fill="#3b2416"/><ellipse class="eye" cx="122" cy="88" rx="8" ry="11" fill="#3b2416"/>
<circle cx="60" cy="110" r="9" fill="#ffb7b7" opacity=".75"/><circle cx="140" cy="110" r="9" fill="#ffb7b7" opacity=".75"/>
<path d="M82 112Q100 134 118 112" stroke="#3b2416" stroke-width="5" fill="none" stroke-linecap="round"/>
<path class="tw2" d="M182 28l4 10 10 4-10 4-4 10-4-10-10-4 10-4z" fill="#e8a020"/><path class="tw2" style="animation-delay:.5s" d="M14 60l3 8 8 3-8 3-3 8-3-8-8-3 8-3z" fill="#c98a4b"/></svg>
<h2 class="brand">Happy Dent</h2>
<?php if($flash)echo"<div class=flash>".h($flash)."</div>";?>
<div class="tabs"><button type="button" class="on" onclick="tab(this,'patient')">🧑 Patient</button><button type="button" onclick="tab(this,'doctor')">🩺 Doctor</button><button type="button" onclick="tab(this,'staff')">👥 Staff</button></div>
<form method="post" id="lf"><input type="hidden" name="action" value="login"><input type="hidden" name="as" id="as" value="patient">
<div class="fld"><input name="email" id="em" type="email" placeholder="Email" value="kavya@gmail.com" required></div>
<div class="fld"><input name="password" type="password" placeholder="Password" value="admin123" required onfocus="this.closest('.lc').classList.add('shy')" onblur="this.closest('.lc').classList.remove('shy')"></div>
<button class="btn" style="width:100%">Login 😁</button>
<p id="pr" style="margin-top:12px"><a href="#" id="prl" onclick="reg(1);return false" style="color:var(--gd);font-weight:700">New patient? Register here →</a></p>
<p style="font-size:12px;color:var(--mu);margin-top:8px">Demo password: admin123</p></form>
<form method="post" id="rf" style="display:none"><input type="hidden" name="action" value="register"><input type="hidden" name="as" id="ras" value="patient">
<div class="fld"><input name="name" placeholder="Full name" required></div><div class="fld"><input name="phone" placeholder="Mobile number" required></div>
<div class="rp" style="display:flex;gap:8px"><div class="fld" style="flex:1"><input name="age" type="number" placeholder="Age" required></div><div class="fld" style="flex:1"><select name="gender" style="width:100%;padding:11px;border-radius:12px;border:1px solid var(--line)"><option>Male</option><option>Female</option><option>Other</option></select></div></div>
<div class="rd" style="display:none"><div class="fld"><input name="spec" placeholder="Specialization (e.g. Orthodontist)" disabled required></div>
<div class="fld"><select name="hospital_id" disabled style="width:100%;padding:11px;border-radius:12px;border:1px solid var(--line)"><?php foreach(opts('hospitals') as $i=>$n)echo"<option value=$i>".h($n)."</option>";?></select></div></div>
<div class="fld"><input name="email" type="email" placeholder="Email" required></div>
<div class="fld"><input name="password" type="password" minlength="6" placeholder="Create password" required onfocus="this.closest('.lc').classList.add('shy')" onblur="this.closest('.lc').classList.remove('shy')"></div>
<button class="btn g" style="width:100%">Create Account 🎉</button><p style="margin-top:12px"><a href="#" onclick="reg(0);return false" style="color:var(--gd);font-weight:700">← Back to login</a></p></form></div>
<script>
function tab(b,t){document.querySelectorAll('.tabs button').forEach(x=>x.classList.remove('on'));b.classList.add('on');document.getElementById('as').value=t;document.getElementById('ras').value=t;
 document.getElementById('em').value={patient:'kavya@gmail.com',doctor:'arun@happydent.com',staff:'admin@happydent.com'}[t];
 const P=t==='patient',D=t==='doctor';
 document.querySelector('.rp').style.display=P?'flex':'none';document.querySelectorAll('.rp input,.rp select').forEach(x=>x.disabled=!P);
 document.querySelector('.rd').style.display=D?'':'none';document.querySelectorAll('.rd input,.rd select').forEach(x=>x.disabled=!D);
 document.getElementById('prl').textContent={patient:'New patient? Register here →',doctor:'New doctor? Register here →',staff:'New staff? Register here →'}[t]}
function reg(x){document.getElementById('lf').style.display=x?'none':'';document.getElementById('rf').style.display=x?'':'none';document.querySelector('.tabs').style.display=x?'none':''}
</script></div>
<?php else: if(!can($p))$p='dashboard'; ?>
<aside class="side" id="side"><div class="logo"><i>🦷</i>Happy Dent</div>
<?php foreach($MENU as $k=>$m)if(can($k))echo"<a href='?p=$k' class='".($p===$k?'on':'')."'><span>{$m[0]}</span>{$m[1]}</a>";?>
<div class="me"><b><?=h($u['name'])?></b><?=h($u['role'])?> · <a href="?logout" style="color:var(--gd)">Logout</a></div></aside>
<div class="main"><div class="top"><button class="burger" onclick="document.getElementById('side').classList.toggle('on')">☰</button><h1><?=$MENU[$p][1]?></h1><div class="sp"></div>
<input class="search" placeholder="🔍 Search this page..." oninput="flt(this.value)"><div class="tpw"><button class="ic" title="Theme colour" onclick="document.getElementById('tp').classList.toggle('on')">🎨</button><div class="tp" id="tp"><?=sw()?></div></div><span class="date">Today · <?=date('d M Y')?></span></div>
<div class="wrap" onclick="document.getElementById('side').classList.remove('on')">
<?php if($flash)echo"<div class=flash>".h($flash)."</div>";

function cell($k,$v){if($k==='dn'&&!$v)return '<i style="color:var(--mu)">Awaiting doctor</i>';if($k==='status')return badge($v);if($k==='amount')return '₹'.number_format($v);if($k==='adate'||$k==='idate')return date('d M Y',strtotime($v));if($k==='atime')return date('h:i A',strtotime($v));if($k==='created')return date('d M Y',strtotime($v));return h($v);}

if($p==='dashboard'){
 $pt=$R==='Patient';$col=['Pending'=>'#c9a227','Visited'=>'#2f9e5b','Booked'=>'#4a7fd4','Unvisited'=>'#e8a020','Postponed'=>'#8a5cc4','Skipped'=>'#d94a4a','Cancelled'=>'#aaa'];
 $up="<div class=card><h3>Upcoming Appointments</h3><div class=tw><table><tr><th>Date<th>Time<th>Patient<th>Doctor<th>Hospital<th>Status</tr>";
 foreach(q("SELECT a.*,p.name pn,d.name dn,h.name hn FROM appointments a JOIN patients p ON p.id=a.patient_id LEFT JOIN doctors d ON d.id=a.doctor_id LEFT JOIN hospitals h ON h.id=a.hospital_id WHERE adate>=CURDATE()$df ORDER BY adate,atime LIMIT 8",$dp) as $r)
  $up.="<tr><td>".cell('adate',$r['adate'])."<td>".cell('atime',$r['atime'])."<td>".h($r['pn'])."<td>".cell('dn',$r['dn'])."<td>".h($r['hn'])."<td>".badge($r['status'])."</tr>";
 $up.="</table></div></div>";
 if($pt){$n=q("SELECT COUNT(*) t,SUM(adate>=CURDATE() AND status IN('Booked','Pending')) u,SUM(status='Visited') v FROM appointments a WHERE 1=1$df",$dp)->fetch();$pb=q("SELECT COALESCE(SUM(amount),0) FROM invoices WHERE status='Pending' AND patient_id=?",$dp)->fetchColumn();
  echo"<div class=hero><div><h2>Welcome, ".h($u['name'])." 👋</h2><p>Book your visit at Keerthana Hospital or Happy Dent Hospital – Thanjavur.</p></div><a class=btn href='?p=book'>➕ Book Appointment</a></div><div class=stats><div class=stat><small>My Appointments</small><b>{$n['t']}</b></div><div class=stat><small>Upcoming</small><b>".(int)$n['u']."</b></div><div class=stat><small>Visited</small><b>".(int)$n['v']."</b></div><div class=stat><small>Pending Bills</small><b>₹".number_format($pb)."</b></div></div>".$up;}
 else{$tp=q("SELECT COUNT(*) FROM patients")->fetchColumn();$cnt=array_fill_keys($AS,0);
  foreach(q("SELECT status,COUNT(*) c FROM appointments a WHERE adate=CURDATE()$df GROUP BY status",$dp) as $r)$cnt[$r['status']]=(int)$r['c'];$tot=array_sum($cnt)?:1;$acc=0;$gr=[];
  foreach($cnt as $k=>$c){$nn=$acc+$c/$tot*100;$gr[]="{$col[$k]} {$acc}% {$nn}%";$acc=$nn;}
  echo"<div class=stats><div class=stat><small>Total Patients</small><b>$tp</b></div><div class=stat><small>Today's Appointments</small><b>".array_sum($cnt)."</b></div>";
  foreach(['Pending','Visited','Unvisited','Postponed','Skipped'] as $k)echo"<div class=stat style='border-color:{$col[$k]}'><small>$k</small><b>{$cnt[$k]}</b></div>";
  echo"</div><div class=grid2><div class=card><h3>Appointment Status (Today)</h3><div class=donut style='background:conic-gradient(".implode(',',$gr).")'><div><span><b>".array_sum($cnt)."</b>Appointments</span></div></div><div class=lg>";
  foreach($cnt as $k=>$c)echo"<span><i style='background:{$col[$k]}'></i>$k $c</span>";
  echo"</div></div>".$up."</div>";}
}
elseif(isset($E[$p])){$e=$E[$p];$W='';$WP=[];if($p==='appointments'&&$df){$W='WHERE 1=1'.$df;$WP=$dp;}if($p==='invoices'&&$R==='Patient'){$W='WHERE i.patient_id=?';$WP=$dp;}
 $rows=q(str_replace('{W}',$W,$e['q']),$WP)->fetchAll();
 echo"<div class=bar>".($R==='Patient'?"<a class=btn href='?p=book'>➕ Book Appointment</a>":"<button class=btn onclick='openM()'>＋ Add New</button>")."</div><div class=card><div class=tw><table id=tb><tr>";
 foreach($e['show'] as $l)echo"<th>$l";echo"<th></tr>";
 foreach($rows as $r){echo"<tr>";foreach($e['show'] as $k=>$l)echo"<td>".cell($k,$r[$k]??'')."</td>";
  $j=$r;unset($j['password']);
  echo"<td class=act>";
  if($R==='Patient'){if($p==='appointments'&&in_array($r['status'],['Booked','Pending']))echo"<form method=post onsubmit=\"return confirm('Cancel appointment?')\"><input type=hidden name=action value=cancel><input type=hidden name=id value={$r['id']}><button class='btn o' style='padding:5px 10px'>Cancel</button></form>";echo"</td></tr>";continue;}
  if($p==='appointments'&&$R==='Doctor'&&$r['status']==='Pending'&&!$r['doctor_id']){echo"<form method=post onsubmit=\"return confirm('Accept this patient?')\"><input type=hidden name=action value=accept><input type=hidden name=id value={$r['id']}><button class='btn g' style='padding:5px 10px'>✅ Accept</button></form></td></tr>";continue;}
  if($p==='appointments')echo"<form method=post style='display:inline'><input type=hidden name=action value=st><input type=hidden name=id value={$r['id']}><select name=status onchange='this.form.submit()' style='padding:4px;border-radius:6px;border:1px solid #ddd'>".implode('',array_map(fn($s)=>"<option".($s===$r['status']?' selected':'').">$s</option>",$AS))."</select></form> ";
  if($p==='invoices'&&$r['status']==='Pending')echo"<form method=post style='display:inline'><input type=hidden name=action value=save><input type=hidden name=id value={$r['id']}><input type=hidden name=patient_id value={$r['patient_id']}><input type=hidden name=amount value={$r['amount']}><input type=hidden name=idate value={$r['idate']}><input type=hidden name=status value=Paid><button class=ic title='Mark paid'>✅</button></form>";
  echo"<button class=ic data-r='".h(json_encode($j))."' onclick='editM(this)'>✏️</button><form method=post onsubmit=\"return confirm('Delete this record?')\"><input type=hidden name=action value=del><input type=hidden name=id value={$r['id']}><button class=ic>🗑️</button></form></td></tr>";}
 echo"</table></div></div>
 <div class=modal id=m><form class=mb method=post><h3 id=mt>{$e['t']}</h3><input type=hidden name=action value=save><input type=hidden name=id id=id>";
 foreach($e['f'] as $k=>$f){echo"<div class=fld><label>{$f[0]}</label>";
  if($f[1]==='sel'){echo"<select name=$k>".($p==='appointments'&&$k==='doctor_id'?"<option value=''>— Not assigned (doctor will accept) —</option>":"");foreach(opts($f[2]) as $ov=>$ol)echo"<option value='".h($ov)."'>".h($ol)."</option>";echo"</select>";}
  elseif($f[1]==='area')echo"<textarea name=$k rows=3></textarea>";
  else echo"<input type={$f[1]} name=$k".(in_array($k,['name','phone','adate','atime','amount','patient_id'])?' required':'').">";
  echo"</div>";}
 echo"<div style='display:flex;gap:10px;justify-content:flex-end'><button type=button class='btn o' onclick='closeM()'>Cancel</button><button class=btn>Save</button></div></form></div>";
}
elseif($p==='calendar'){
 $m=$_GET['m']??date('Y-m');$f=strtotime($m.'-01');$dn=(int)date('t',$f);$off=(int)date('w',$f);$by=[];
 foreach(q("SELECT a.*,p.name pn,d.name dn FROM appointments a JOIN patients p ON p.id=a.patient_id LEFT JOIN doctors d ON d.id=a.doctor_id WHERE DATE_FORMAT(adate,'%Y-%m')=?$df ORDER BY atime",[$m,...$dp]) as $r){$r['atime']=date('h:i A',strtotime($r['atime']));$by[$r['adate']][]=$r;}
 $col=['Pending'=>'#c9a227','Visited'=>'#2f9e5b','Booked'=>'#4a7fd4','Unvisited'=>'#e8a020','Postponed'=>'#8a5cc4','Skipped'=>'#d94a4a','Cancelled'=>'#aaa'];
 echo"<div class=grid2 style='grid-template-columns:1.8fr 1fr'><div class=card><div class=bar><a class='btn o' href='?p=calendar&m=".date('Y-m',strtotime('-1 month',$f))."'>‹</a><b style='flex:1;text-align:center'>".date('F Y',$f)."</b><a class='btn o' href='?p=calendar&m=".date('Y-m',strtotime('+1 month',$f))."'>›</a></div><div class=cal>";
 foreach(['Sun','Mon','Tue','Wed','Thu','Fri','Sat'] as $d)echo"<div class=h>$d</div>";
 for($i=0;$i<$off;$i++)echo"<div></div>";
 for($d=1;$d<=$dn;$d++){$k=sprintf('%s-%02d',$m,$d);echo"<div class='d".($k===date('Y-m-d')?' today':'')."' onclick=\"day('$k',this)\">$d<br>";foreach($by[$k]??[] as $r)echo"<span class=dot style='background:{$col[$r['status']]}'></span>";echo"</div>";}
 echo"</div><div class=lg style='margin-top:12px'>";foreach($col as $k=>$c)echo"<span><i style='background:$c'></i>$k</span>";
 echo"</div></div><div class=card><h3 id=dt>Select a date</h3><div id=dl style='color:var(--mu)'>Tap a day to see appointments.</div></div></div><script>const CAL=".json_encode($by).";</script>";
}
elseif($p==='reports'){
 $fr=$_GET['from']??date('Y-m-01');$to=$_GET['to']??date('Y-m-d');$rows=q("SELECT adate d,COUNT(*) t,SUM(status='Visited') v FROM appointments WHERE adate BETWEEN ? AND ? GROUP BY adate ORDER BY adate",[$fr,$to])->fetchAll();$mx=max(1,...array_column($rows,'t')?:[1]);
 $rv=q("SELECT SUM(CASE WHEN status='Paid' THEN amount END) p,SUM(CASE WHEN status='Pending' THEN amount END) n FROM invoices WHERE idate BETWEEN ? AND ?",[$fr,$to])->fetch();$pd=(float)$rv['p'];$pn=(float)$rv['n'];$pc=$pd+$pn?round($pd/($pd+$pn)*100):0;
 echo"<form class=bar method=get><input type=hidden name=p value=reports><input type=date name=from value=$fr class=search style='max-width:170px'><input type=date name=to value=$to class=search style='max-width:170px'><button class=btn>Filter</button><a class='btn g' href='?p=reports&csv=1&from=$fr&to=$to'>⬇ Export CSV</a></form>
 <div class=grid2 style='grid-template-columns:1.6fr 1fr'><div class=card><h3>Patient Visits</h3><div class=chart>";
 foreach($rows as $r){echo"<div class=c><div style='height:".($r['t']/$mx*140)."px;background:#e8a020;border-radius:4px 4px 0 0'></div><div style='height:".($r['v']/$mx*140)."px;background:#2f9e5b'></div>".date('d M',strtotime($r['d']))."</div>";}
 if(!$rows)echo"<p>No data in range.</p>";
 echo"</div><div class=lg><span><i style='background:#2f9e5b'></i>Visited</span><span><i style='background:#e8a020'></i>Total booked</span></div></div>
 <div class=card><h3>Revenue</h3><div class=donut style='background:conic-gradient(#2f9e5b 0 {$pc}%,#e8a020 {$pc}% 100%)'><div><span><b>₹".number_format($pd+$pn)."</b>Total</span></div></div><div class=lg><span><i style='background:#2f9e5b'></i>Paid ₹".number_format($pd)."</span><span><i style='background:#e8a020'></i>Pending ₹".number_format($pn)."</span></div></div></div>";
}
elseif($p==='whatsapp'){
 echo"<div class=grid2 style='grid-template-columns:1fr 1.4fr'><div class=card><h3>Send Message</h3><form method=post><input type=hidden name=action value=wa><div class=fld><label>Patient</label><select name=pid>";
 foreach(opts('patients') as $i=>$n)echo"<option value=$i>".h($n)."</option>";
 echo"</select></div><div class=fld><label>Message</label><textarea name=msg rows=5 required>Hello, this is a reminder from Happy Dent for your upcoming dental appointment. 😊</textarea></div><button class='btn g'>Log Message</button></form></div>
 <div class=card><h3>WhatsApp Log (auto-added on every new appointment)</h3><div class=chat>";
 foreach(q("SELECT * FROM whatsapp_logs ORDER BY id DESC LIMIT 30") as $r)echo"<div class=msg>".h($r['msg'])."<small>To {$r['phone']} · {$r['created']} · <a target=_blank style='color:#1f7a44;font-weight:600' href='https://wa.me/91{$r['phone']}?text=".urlencode($r['msg'])."'>Open WhatsApp ↗</a></small></div>";
 echo"</div></div></div>";
}
elseif($p==='book'){
 $H=q("SELECT * FROM hospitals")->fetchAll();$hid=(int)($_GET['h']??0);$dt=$_GET['dt']??'';
 echo"<div class=card><h3>1️⃣ Choose Hospital</h3><div class=hosp>";
 foreach($H as $x)echo"<a class='hc".($hid==$x['id']?' on':'')."' href='?p=book&h={$x['id']}'><span>🏥</span><b>".h($x['name'])."</b><small>".h($x['area'])."<br>".h($x['address'])."</small></a>";
 echo"</div></div>";
 if($hid){
  $hs=implode(' · ',array_map(fn($r)=>date('d M',strtotime($r['hdate'])).' ('.h($r['reason']).')',q("SELECT hdate,reason FROM holidays WHERE hdate>=CURDATE() AND doctor_id=0 ORDER BY hdate LIMIT 6")->fetchAll()))?:'none';
  echo"<div class=card><h3>2️⃣ Pick Date</h3><input type=date class=search value='".h($dt)."' min='".date('Y-m-d')."' max='".date('Y-m-d',strtotime('+30 day'))."' onchange=\"location='?p=book&h=$hid&dt='+this.value\"><p style='margin-top:10px;color:var(--mu)'>🚫 Holidays: $hs</p></div>";
  if($dt&&$dt>=date('Y-m-d')&&($hr=isHol($dt,0))!==null)echo"<div class=card style='border-left:6px solid #d94a4a'><h3>🚫 Closed on ".date('d M Y',strtotime($dt))."</h3><p>".h($hr)." – booking is not available. Please pick another date.</p></div>";
  elseif($dt&&$dt>=date('Y-m-d')){
   $cap=cap($hid,$dt);$tk=[];
   foreach(q("SELECT LEFT(atime,5) t,COUNT(*) c FROM appointments WHERE hospital_id=? AND adate=? AND status NOT IN('Cancelled','Skipped') GROUP BY LEFT(atime,5)",[$hid,$dt]) as $r)$tk[$r['t']]=(int)$r['c'];
   if(!$cap)echo"<div class=card><p>No doctors available on this date. Please pick another day.</p></div>";
   else{
   echo"<form class=card method=post><h3>3️⃣ Available Slots – ".date('d M Y',strtotime($dt))."</h3><input type=hidden name=action value=book><input type=hidden name=h value=$hid><input type=hidden name=dt value='".h($dt)."'><div class=slots>";$any=0;
   for($t=540;$t<1020;$t+=30){$sl=sprintf('%02d:%02d',intdiv($t,60),$t%60);if($t>=780&&$t<840)continue;$off=(($tk[$sl]??0)>=$cap)||($dt===date('Y-m-d')&&$sl<=date('H:i'));
    echo"<label class='slot".($off?' off':'')."'><input type=radio name=tm value=$sl ".($off?'disabled':'required')."><span>".date('h:i A',strtotime($sl))."</span></label>";if(!$off)$any=1;}
   echo"</div>".($any?"<div class=fld style='margin-top:14px'><label>Reason for visit (a free doctor will accept your request)</label><input name=reason placeholder='Tooth pain, cleaning...' required></div><button class=btn>✅ Send Request</button>":"<p style='margin-top:12px'>No free slots on this date. Try another day.</p>")."</form>";}}
 }
}
elseif($p==='attendance'){
 $all=$R==='Doctor'
  ?q("SELECT 'doctor' ut,d.id,d.name,'Doctor' role,a.check_in,a.check_out FROM doctors d LEFT JOIN attendance a ON a.staff_id=d.id AND a.utype='doctor' AND a.adate=CURDATE() WHERE d.id=?",[$u['id']])->fetchAll()
  :q("SELECT 'staff' ut,s.id,s.name,s.role,a.check_in,a.check_out FROM staff s LEFT JOIN attendance a ON a.staff_id=s.id AND a.utype='staff' AND a.adate=CURDATE() WHERE s.id=?",[$u['id']])->fetchAll();
 $ob=['Doctor'=>'🩺','Receptionist'=>'🎧','Admin'=>'🛡️','Staff'=>'🪥'];
 echo"<div class=card style='display:flex;align-items:center;gap:16px;flex-wrap:wrap'><div class=clock id=ck>".date('h:i:s A')."</div><div><b>".date('l, d M Y')."</b><br><small style='color:var(--mu)'>Check in when you arrive · check out when you leave</small></div></div><div class=att>";
 foreach($all as $r){$ci=$r['check_in'];$co=$r['check_out'];$st=!$ci?'out0':(!$co?'in':'done');
  $wk=($ci&&$co)?floor((strtotime($co)-strtotime($ci))/3600).'h '.floor(((strtotime($co)-strtotime($ci))%3600)/60).'m':'';
  $fm="<form method=post><input type=hidden name=action value=att><input type=hidden name=sid value={$r['id']}><input type=hidden name=ut value={$r['ut']}>";
  echo"<div class='ac $st'><div class=ob><span>".($ob[$r['role']]??'🦷')."</span></div><b>".h($r['name'])."</b><small>".h($r['role'])."</small><div class=tm><i>🟢 In: ".($ci?date('h:i A',strtotime($ci)):'–')."</i><i>🔴 Out: ".($co?date('h:i A',strtotime($co)):'–')."</i>".($wk?"<i>⏱ Worked: $wk</i>":'')."</div>";
  if(!$ci)echo"$fm<input type=hidden name=type value=in><button class='btn ci' style='width:100%'>🚪 Check In</button></form>";
  elseif(!$co)echo"$fm<input type=hidden name=type value=out><button class='btn co' style='width:100%'>🏃 Check Out</button></form>";
  else echo"<span class='b b-Visited'>✅ Day completed</span>";
  echo"</div>";}
 echo"</div>";
}
elseif($p==='settings'){
 echo"<div class=card><h3>🎨 Theme Colour</h3><p style='color:var(--mu);margin-bottom:12px'>Click a colour – the whole site changes instantly.</p>".sw()."</div>";
 echo"<div class=card style='max-width:440px'><h3>🔑 My Account</h3><p style='margin-bottom:12px;color:var(--mu)'>".h($u['name'])." · ".h($u['role'])." · ".h($u['email'])."</p><form method=post><input type=hidden name=action value=pw><div class=fld><label>New Password</label><input type=password name=pw minlength=6 required></div><button class=btn>Change Password</button></form></div>";
 if(in_array($R,['Admin','Receptionist','Doctor'])){
  $m=$_GET['m']??date('Y-m');if(!preg_match('/^\d{4}-\d\d$/',$m))$m=date('Y-m');$f=strtotime($m.'-01');$dn=(int)date('t',$f);$off=(int)date('w',$f);$o=$R==='Doctor'?(int)$u['id']:0;$H=[];
  foreach(q("SELECT * FROM holidays WHERE DATE_FORMAT(hdate,'%Y-%m')=? AND doctor_id IN(0,?)",[$m,$o]) as $r)$H[$r['hdate']]=$r;
  echo"<div class=card><h3>🗓️ ".($R==='Doctor'?'My Leave Calendar':'Clinic Holiday Calendar')."</h3><p style='color:var(--mu);margin-bottom:12px'>Type a reason, then click a date to mark it as holiday. Click again to remove. Patients cannot book on these dates.</p>
  <form id=hf method=post class=bar><input type=hidden name=action value=hol><input type=hidden name=d id=hd><input name=reason class=search style='max-width:320px' placeholder='Reason (e.g. Diwali, Doctor leave)'></form>
  <div class=bar><a class='btn o' href='?p=settings&m=".date('Y-m',strtotime('-1 month',$f))."'>‹</a><b style='flex:1;text-align:center'>".date('F Y',$f)."</b><a class='btn o' href='?p=settings&m=".date('Y-m',strtotime('+1 month',$f))."'>›</a></div><div class=cal>";
  foreach(['Sun','Mon','Tue','Wed','Thu','Fri','Sat'] as $d)echo"<div class=h>$d</div>";
  for($i=0;$i<$off;$i++)echo"<div></div>";
  for($d=1;$d<=$dn;$d++){$k=sprintf('%s-%02d',$m,$d);$x=$H[$k]??null;$lk=$x&&$x['doctor_id']!=$o;
   echo"<div class='d".($x?' hol':'').($lk?' lock':'')."'".($lk?'':" onclick=\"hol('$k')\"").">$d".($x?"<small>".($lk?'🔒':'🚫')." ".h($x['reason'])."</small>":'')."</div>";}
  echo"</div></div>";}
}
?></div></div>
<script>
const $=s=>document.querySelector(s);
function flt(v){v=v.toLowerCase();document.querySelectorAll('#tb tr:not(:first-child)').forEach(r=>r.style.display=r.textContent.toLowerCase().includes(v)?'':'none')}
function openM(){const f=$('#m form');f.reset();$('#id').value='';$('#m').classList.add('on')}
function closeM(){$('#m').classList.remove('on')}
function editM(b){const r=JSON.parse(b.dataset.r),f=$('#m form');f.reset();$('#id').value=r.id;for(const k in r){if(f.elements[k])f.elements[k].value=r[k]??''}$('#m').classList.add('on')}
function day(k,el){document.querySelectorAll('.cal .d').forEach(x=>x.classList.remove('sel'));el.classList.add('sel');$('#dt').textContent=new Date(k).toDateString();
 const a=CAL[k]||[];$('#dl').innerHTML=a.length?a.map(r=>`<div style="padding:8px 0;border-bottom:1px solid #eee"><b>${r.atime}</b> · ${r.pn}<br><small>${r.dn||'Doctor not assigned'} · ${r.reason}</small> <span class="b b-${r.status}">${r.status}</span></div>`).join(''):'No appointments.'}
function hol(d){$('#hd').value=d;$('#hf').submit()}
setInterval(()=>{const c=$('#ck');if(c)c.textContent=new Date().toLocaleTimeString()},1000);
document.addEventListener('keydown',e=>{if(e.key==='Escape'&&$('#m'))closeM()});
</script>
<?php endif; ?></body></html>
