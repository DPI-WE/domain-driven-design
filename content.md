# Embracing Complexity with Domain-Driven Design 💡
Having explored the utility and elegance of Service Objects, it's time to dive deeper and discover the transformative power of Domain-Driven Design. This approach will not only enrich your understanding of complex business logic but also teach you how to encapsulate this logic within rich domain models, elevating your Rails applications to new heights of clarity and maintainability.

## The Essence of Domain-Driven Design
At its core, Domain-Driven Design is about understanding and reflecting the complexity of the business environment through your application's architecture. It focuses on placing the domain and the logic of the business at the forefront of the development process, encouraging developers to speak the same language as domain experts (business analysts, stakeholders, etc.).

### Rich Domain Models: More Than Just Data
In the context of Domain-Driven Design, a domain model isn't just a representation of data in your database; it's a living blueprint of the business logic. These models should encapsulate the behaviors, rules, and logic that define the domain, making them rich in functionality and not just data storage entities.

### Crafting Rich Domain Models
Let's put theory into practice by enhancing a Rails model with behaviors and logic that reflect the intricacies of our domain. Suppose we're building a platform for managing events. An event in our system has complexities beyond just its date, location, and attendees. It involves scheduling, capacity management, and notifications.

1. Defining Behaviors in the Model
First, let's enrich our Event model by adding business logic directly related to events.

```ruby
# app/models/event.rb
class Event < ApplicationRecord
  has_many :attendees
  validates :name, :date, :location, presence: true

  def reschedule(new_date)
    # Business logic to reschedule an event
    self.date = new_date
    notify_attendees
  end

  def add_attendee(user)
    # Logic to add an attendee, considering capacity constraints
    return false if attendees.count >= capacity
    attendees << user
    true
  end

  private

  def notify_attendees
    # Logic to notify all attendees about the event rescheduling
    attendees.each do |attendee|
      # Send notification to attendee
    end
  end
end
```

2. Interacting with the Enhanced Model
Now, with our rich domain model, actions like rescheduling an event or adding an attendee encapsulate all necessary checks, validations, and side effects (e.g., notifications).

```ruby
# Rescheduling an event
event = Event.find(params[:id])
event.reschedule(params[:new_date])

# Adding an attendee to an event
event.add_attendee(current_user)
```

## The Benefits of Rich Domain Models
- **Encapsulation**: Business logic is kept where it belongs, within the domain model itself, leading to cleaner controllers and services.
- **Ubiquitous Language**: The model reflects the language and complexity of the business domain, making it easier for developers and domain experts to communicate.
- **Flexibility**: Business rules and behaviors encapsulated in the model can be easily modified or extended, enhancing the application's ability to evolve with the business.

## Refactor Order Processing Logic into a Domain Model
Let's refactor the business logic for order processing, currently embedded in a controller, into the Order domain model. This will demonstrate the encapsulation of business logic within the domain model, making the controller leaner and the model richer—a key practice in Domain-Driven Design.

### Initial Controller Code

```ruby
# app/controllers/orders_controller.rb
class OrdersController < ApplicationController
  def create
    @order = Order.new(order_params)
    if @order.save
      # TODO: move this business logic to the model layer
      if @order.total_price >= 100
        @order.update(discount_applied: true, discount_amount: 10)
      end
      redirect_to order_path(@order), notice: 'Order was successfully created.'
    else
      render :new
    end
  end

  private

  def order_params
    params.require(:order).permit(:customer_id, :total_price, :discount_applied, :discount_amount)
  end
end
```

### Task: Refactor to Order Model
Now we'll refactor the order discount logic into the Order model. The Order model should automatically apply a discount if certain conditions are met, encapsulating this business rule within the model itself.

<aside>
  You may want to use an [active record callback](https://guides.rubyonrails.org/active_record_callbacks.html) to automatically apply a discount before saving a record.
</aside>

```ruby
# app/models/order.rb
class Order < ApplicationRecord
  before_save :apply_discount

  private

  def apply_discount
    if total_price >= 100 && !discount_applied?
      self.discount_applied = true
      self.discount_amount = 10 # Assume 10 is the discount amount for the simplicity
    end
  end
end
```
<!-- TODO: make it a coding exercise later when we can use ActiveRecord
{: .repl #order title="Order Model" readonly_lines="[1,2,3,4,5]"}

```ruby
# spec/models/order_spec.rb
RSpec.describe Order, type: :model do
  it 'applies a discount on orders over $100' do
    order = Order.create(customer_id: 1, total_price: 150)
    expect(order.discount_applied).to be true
    expect(order.discount_amount).to eq(10)
  end
```
{: .repl-test #order_model_test_1 for="order" title="Order Model applies discounts correctly > 100" points="2" }

```ruby
RSpec.describe Order, type: :model do
  it 'does not apply a discount on orders under $100' do
    order = Order.create(customer_id: 1, total_price: 99)
    expect(order.discount_applied).to be false
    expect(order.discount_amount).to be_nil
  end
end
```
{: .repl-test #order_model_test_2 for="order" title="Order Model applies discounts correctly < 100>" points="2" }

-->

Now our `OrdersController` can simply save an `Order` without having to apply discounts (or be concerned with any new business logic we add later).

```ruby
# app/controllers/orders_controller.rb
class OrdersController < ApplicationController
  def create
    @order = Order.new(order_params)
    if @order.save
      redirect_to order_path(@order), notice: 'Order was successfully created.'
    else
      render :new
    end
  end

  private

  def order_params
    params.require(:order).permit(:customer_id, :total_price, :discount_applied, :discount_amount)
  end
end
```

By encapsulating business rules within the model, we create a more maintainable and understandable application architecture. This not only simplifies the controller but also aligns the model more closely with the business domain it represents.

## Quiz

- What is the primary goal of Domain-Driven Design?
- To simplify the coding process by reducing the number of classes and methods.
  - Not quite. Domain-Driven Design focuses more on aligning the software design with business needs, not just simplifying code structure.
- To ensure the application speaks the same language as domain experts.
  - Correct! Domain-Driven Design aims to create a common language between developers and domain experts to enhance understanding and communication.
- To facilitate faster web application rendering.
  - Incorrect. Domain-Driven Design does not specifically target rendering performance; it focuses on business and domain complexities.
- To minimize the use of databases in applications.
  - Incorrect. Domain-Driven Design does not primarily concern database use but rather the alignment of design with business concepts.
{: .choose_best #ddd_primary_goal title="Primary Goal of Domain-Driven Design" points="1" answer="2" }

- What role do rich domain models play in Domain-Driven Design?
- They serve as simple data containers.
  - Incorrect. Rich domain models are more than just data containers; they encapsulate complex business logic and behaviors.
- They contain validations and relationships but no behaviors.
  - Incorrect. Rich domain models in Domain-Driven Design should include behaviors, not just data relationships and validations.
- They encapsulate behaviors, rules, and logic of the domain.
  - Correct! Rich domain models are central to Domain-Driven Design as they embody the complex logic and rules of the domain.
- They are solely used for data migration.
  - Incorrect. Their role is much broader and focuses on representing the domain comprehensively.
{: .choose_best #ddd_role_models title="Role of Rich Domain Models" points="1" answer="3" }

- Why is encapsulation important in Domain-Driven Design?
- It enhances the graphical interface of the application.
  - Incorrect. Encapsulation in Domain-Driven Design is about maintaining business logic within domain models, not about UI aspects.
- It keeps business logic within the domain model, leading to cleaner and more maintainable code.
  - Correct! Encapsulation helps keep the domain model focused on the core business logic, enhancing clarity and maintainability.
- It only allows the use of external APIs.
  - Incorrect. This answer misrepresents the purpose of encapsulation in the context of Domain-Driven Design.
- It restricts the application to a single database model.
  - Incorrect. Encapsulation in Domain-Driven Design does not concern database models directly but rather the containment of business logic.
{: .choose_best #ddd_importance_encapsulation title="Importance of Encapsulation in Domain-Driven Design" points="1" answer="2" }

- What is ubiquitous language in the context of Domain-Driven Design?
- A programming language that must be used in all domain-driven applications.
  - Incorrect. Ubiquitous language refers to common terminology, not a specific programming language.
- A common language shared between developers and domain experts that improves communication and understanding.
  - Correct! The ubiquitous language facilitates effective communication by using consistent terminology across the project.
- The documentation language that must be used throughout the application.
  - Incorrect. It's about common language in discussions and design, not just documentation.
- A specific set of technical jargons used in database design.
  - Incorrect. Ubiquitous language covers all domain aspects, not just database-related terms.
{: .choose_best #ddd_ubiquitous_language title="Ubiquitous Language in Domain-Driven Design" points="1" answer="2" }

- What is one of the benefits of using Domain-Driven Design in software development?
- Reduces the need for testing the application.
  - Incorrect. Domain-Driven Design does not reduce the need for testing; if anything, it underscores the importance of thorough testing aligned with business logic.
- Increases the complexity of the codebase.
  - Incorrect. While Domain-Driven Design handles complex domains, it aims to make the code more manageable and aligned with business needs, not unnecessarily complex.
- Allows for easy modification and extension of business rules.
  - Correct! One of the key benefits of Domain-Driven Design is that it facilitates the adaptability of the software to evolving business requirements.
- Eliminates the need for user feedback.
  - Incorrect. Domain-Driven Design does not eliminate the need for user feedback; user insights are crucial for refining the domain model.
{: .choose_best #ddd_benefits title="Benefits of Domain-Driven Design" points="1" answer="3" }

## Conclusion
Adopting Domain-Driven Design encourages a deeper engagement with the business domain, leading to applications that are more aligned with business needs and logic. By building rich domain models, you encapsulate complex business rules and behaviors, creating a more intuitive and maintainable codebase.

Domain-Driven Design is not just about technical mastery; it's a mindset shift towards valuing deep understanding and reflection of the business domain in software. As you continue to explore and implement these concepts, remember that the ultimate goal is to bridge the gap between technology and business, crafting applications that not only work well but also speak the language of the domain they serve.

Dive deep, experiment with these principles, and watch as your applications transform into more robust, domain-driven systems. The path of domain-driven design is challenging but rewarding, opening doors to a new level of software craftsmanship. Happy coding!

## Resources
- [Vanilla Rails is Plenty](https://dev.37signals.com/vanilla-rails-is-plenty/)
- [Domain Driven Boldness](https://dev.37signals.com/domain-driven-boldness/)
- [Domain-Driven Design: Tackling Complexity in the Heart of Software](https://www.oreilly.com/library/view/domain-driven-design-tackling/0321125215/)
