## One-To-One-Bi-Directional(雙向)
>參考網址：https://www.796t.com/content/1544860112.html
>
>MappedBy： 
>* 1、只有@OneToOne，@OneToMany，@ManyToMany上才有MappedBy屬性，ManyToOne不存在該屬性； 
>* 2、MappedBy標籤一定是定義在被擁有方的（被控方），他指向擁有方； 
>* 3、MappedBy的含義，應該理解為，擁有方能夠自動維護跟被擁有方的關係，當然，如果從被擁有方，通過手工強行來維護擁有方的關係也是可以做到的； 
>* 4、MappedBy跟joinColumn/JoinTable總是處於互斥的一方，可以理解為正是由於擁有方的關聯被擁有方的字段存在，擁有方才擁有了被擁有方。
>
>MappedBy這方定義JoinColumn/JoinTable總是失效的，不會建立對應的字段或者表。

**對於One-To-One雙向關係，需在InstructorDetail(副表)類別中加入反向關係，透過MappedBy獲取Instructor類別中的instructorDetail屬性所對應的instructor_detail_id以找到講師項目。**
```
// add new filed for instructor (also add getter/setters)
// add @OneToOne annotation
@OneToOne(mappedBy="instructorDetail", 
		cascade= {CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST,
				CascadeType.REFRESH})
private Instructor instructor;
```

**完整程式碼**
```
@Entity
@Table(name="instructor_detail")
public class InstructorDetail {
	
	// annotate the class as an entity and map to db table
	
	// define the fields
	
	// annotate the fields with db column names
	
	// create constructors
	
	// generate getter/setter methods
	
	// generate toString() method
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="id")
	private int id;
	
	@Column(name="youtube_channel")
	private String youtubeChannel;
	
	@Column(name="hobby")
	private String hobby;
	
	// add new filed for instructor (also add getter/setters)
	// add @OneToOne annotation
	@OneToOne(mappedBy="instructorDetail", 
			cascade= {CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST,
					CascadeType.REFRESH})
	private Instructor instructor;
	
	
	public Instructor getInstructor() {
		return instructor;
	}

	public void setInstructor(Instructor instructor) {
		this.instructor = instructor;
	}

	public InstructorDetail()
	{
		
	}

	public InstructorDetail(String youtubeChannel, String hobby) {
		this.youtubeChannel = youtubeChannel;
		this.hobby = hobby;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getYputubeChannel() {
		return youtubeChannel;
	}

	public void setYputubeChannel(String youtubeChannel) {
		this.youtubeChannel = youtubeChannel;
	}

	public String getHobby() {
		return hobby;
	}

	public void setHobby(String hobby) {
		this.hobby = hobby;
	}

	@Override
	public String toString() {
		return "InstructorDetail [id=" + id + ", youtubeChannel=" + youtubeChannel + ", hobby=" + hobby + "]";
	}
}
```
**以下範例為測試雙向對象資料獲取示例**
```
public class GetInstructorDetailDemo {

	public static void main(String[] args) {
		
		// create session factory
		SessionFactory factory = new Configuration()
				.configure("hibernate.cfg.xml")
				.addAnnotatedClass(Instructor.class)
				.addAnnotatedClass(InstructorDetail.class)
				.buildSessionFactory();
		
		// create session
		Session session = factory.getCurrentSession();
		
		try {  				
			
            // start transaction
            session.beginTransaction();
            
            // get the instructor detail object
            int theId = 2999;
            InstructorDetail tempInstructorDetail = 
            		session.get(InstructorDetail.class, theId);
            
            // print the instructor detail
            System.out.println("tampInstructorDetail: " + tempInstructorDetail);
            
            // print associated instructor
            System.out.println("the associated instrictor: " + 
            					tempInstructorDetail.getInstructor());

            // commit transaction
            session.getTransaction().commit();
            
        } catch (Exception exc) {
            exc.printStackTrace();
        } finally {
        	// handle connection leak issue
        	session.close();
        	
            factory.close();
        }
	}
}
```
**此處為刪除功能示例，須注意在InstructorDetail類別中級聯配置不再是ALL，因此當刪除在InstructorDetail項目時，在主表Instructor中並不會跟著同步刪除，因此須注意刪除副表時，需斷開與主表之間的鏈結，避免造成出錯。**
```
public class DeleteInstructorDetailDemo {

	public static void main(String[] args) {
		
		// create session factory
		SessionFactory factory = new Configuration()
				.configure("hibernate.cfg.xml")
				.addAnnotatedClass(Instructor.class)
				.addAnnotatedClass(InstructorDetail.class)
				.buildSessionFactory();
		
		// create session
		Session session = factory.getCurrentSession();
		
		try {  				
			
            // start transaction
            session.beginTransaction();
            
            // get the instructor detail object
            int theId = 3;
            InstructorDetail tempInstructorDetail = 
            		session.get(InstructorDetail.class, theId);
            
            // print the instructor detail
            System.out.println("tampInstructorDetail: " + tempInstructorDetail);
            
            // print associated instructor
            System.out.println("the associated instrictor: " + 
            					tempInstructorDetail.getInstructor());
            
            // now let's delete the instructor detail
            System.out.println("Deleting tempInstructorDetail: " + tempInstructorDetail);
            
            // remove the associated object reference
            // break bi-directional link
            
            tempInstructorDetail.getInstructor().setInstructorDetail(null);
            
            session.delete(tempInstructorDetail);
            
            // commit transaction
            session.getTransaction().commit();
 
            System.out.println("Done!");
            
        } catch (Exception exc) {
            exc.printStackTrace();
        } finally {
        	// handle connection leak issue
        	session.close();
        	
            factory.close();
        }
	}
}
```
