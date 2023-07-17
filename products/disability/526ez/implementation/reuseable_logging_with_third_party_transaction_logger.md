# Reuseable logging with the ThirdPartyTransactionLogging module


## What is it?
This module can be used to wrap one or more controller methods in logging.  

## Why do I need it?
If you have a controller action that delegates to a third party API, you can / should use this module to wrap that method in applicable logging.

This was created to allow us to log information around third party API interactions
    - in a way that is reuseable
    - without coupling our controllers and models
    - In a way the defines a default set of log data that is considered valueble

### Further Context:
- [Epic](https://app.zenhub.com/workspaces/disability-benefits-experience-team-carbs-6470c8bfffee9809b2634a52/issues/gh/department-of-veterans-affairs/va.gov-team/60952)
- [Ticket](https://app.zenhub.com/workspaces/disability-benefits-experience-team-carbs-6470c8bfffee9809b2634a52/issues/gh/department-of-veterans-affairs/va.gov-team/60944)
- [Resarch and Planning document](https://github.com/department-of-veterans-affairs/va.gov-team/blob/master/products/disability/526ez/engineering_research/error_handling_and_submission_failures/03_third_party_action_logging_POC.md)
- Further Context on the run up to this project [here](https://github.com/department-of-veterans-affairs/va.gov-team/blob/master/products/disability/526ez/engineering_research/error_handling_and_submission_failures/01_overview_of_error_and_logging.md) and [here](https://github.com/department-of-veterans-affairs/va.gov-team/blob/master/products/disability/526ez/engineering_research/error_handling_and_submission_failures/02_loose_ends_and_next_steps.md)

## Where can I use it?
This is designed to work the controller level, i.e. in controller files or concerns that are included into controllers.

## When should I use it?
Ideally you should wrap as little code as possible.  That means you may want to first refactor your controller to isolate your third party interaction in a helper method, then wrap that helper method

## How do I use it?

1. extend the module into your controller.  NOTE: you must use `extend`, as `include` will not work.

wipn-image

2. use the `wrap_with_logging` method to idenitify the correct method to log around. 

wipn-image

2.A the `wrap_with_logging` method will log the following default parameters:
  a. Puma process id
  b. current user's uuid
  c. a generic description of the action being taken
  d. a 'breadcrumbs' value defining the file where the log is called, the calling classname and calling method.  This is purely for debugging purposes.
  e. start time, stop time, duration.

  However you can pass additional parameters to the method to either suplement
  or overwrite these default logs using the key `additional_logs:`

  E.G.

wipn-image

## How does it work?

The `ThirdPartyTransactionLogging::MethodWrapper` module is `extend`ed into the calling class, making the `wrap_with_logging` method available at class instantiation.  

wipn-image

An additional module called `ThirdPartyTransactionLogging::ScopedInstanceMethods` will be `include`d into the instance of the calling class at the time that `wrap_with_logging` is called.  This inclusion-at-instantiation of the helper methods makes the controller scope available to the executing method.  This is necessary to allow access to controller level information, such as `current_user`

wipn-image

## What does it not do?

- This should not be used in models.  If you try and use this at the model level you will get a `NoMethodError` on `current_user`.  This is by design, as allowing our models to know about request level data like `current_user` would break encapsulation principles.
- If your third party transaction is triggered from a model level action, then this wrapping will not be the closest possible wrapper to the interaction.  In this case, you should suplent the useage of the module with logging from within the model (or applicable non-controller layer object).

