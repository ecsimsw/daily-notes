## Persistence context with 용관

``` java 
@Service
public class PersonService {

    private final PersonRepository personRepository;

    public PersonService(PersonRepository personRepository) {
        this.personRepository = personRepository;
    }

    @Transactional
    public void save() {
        personRepository.save(new Person("진환"));
    }

    @Transactional
    public void testByName() {
        Person person1 = personRepository.findByName("진환");  // Select -> Persist
        person1.name = "용관";                                 // Persistence Context

        Person person2 = personRepository.findByName("용관");  // Update -> Select -> Persist
        System.out.println(person1 == person2);               // true

        person1.name = "하이";
        System.out.println(person1 == person2);               // true
        System.out.println(person2.name);                     // 하이
    }

    @Transactional
    public void testById() {
        Person person1 = personRepository.findById(1L).get(); // Select
        person1.name = "용관";                                 // Persistence Context

        Person person2 = personRepository.findById(1L).get(); // Just using cache
        System.out.println(person1 == person2);               // true
        // Flush (update) with transaction commit
    }
}
```
