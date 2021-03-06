Processors
==========

Table of Contents
-----------------

-  `Overview <#overview>`__
-  `Creating a processor <#creating-a-processor>`__
-  `Processor conditions <#processor-conditions>`__
-  `Examples of processor conditions <#examples-of-processor-conditions>`__
-  `Error handling <#error-handling>`__

Overview
--------

A processor is the main element where a business logic of Data API is implemented. Each processor must implement `ProcessorInterface <https://github.com/oroinc/platform/tree/master/src/Oro/Component/ChainProcessor/ProcessorInterface.php>`__ and be registered in the dependency injection container using the ``oro.api.processor`` tag.

Please see `actions <./actions.md>`__ section for more details about where and how processors are used.

Also you can use the `oro:api:debug <./commands.md#oroapidebug>`__ command to see all actions and processors.

Creating a processor
--------------------

To create a new processor just create a class implements `ProcessorInterface <https://github.com/oroinc/platform/tree/master/src/Oro/Component/ChainProcessor/ProcessorInterface.php>`__ and `tag <http://symfony.com/doc/current/book/service_container.html#book-service-container-tags>`__ it with the ``oro.api.processor`` name.

.. code:: php

    <?php

    namespace Acme\Bundle\ProductBundle\Api\Processor;

    use Oro\Component\ChainProcessor\ContextInterface;
    use Oro\Component\ChainProcessor\ProcessorInterface;

    /**
     * A short description what the processor does.
     **/
    class DoSomething implements ProcessorInterface
    {
        /**
         * {@inheritdoc}
         */
        public function process(ContextInterface $context)
        {
            // do some work here
        }
    }

.. code:: yaml

    services:
        acme.api.processor.do_something:
            class: Acme\Bundle\ProductBundle\Api\Processor\DoSomething
            tags:
                - { name: oro.api.processor, action: get, group: normalize_input, priority: 10 }

Please note that:

-  The name of a processor usually starts with a verb and the ``Processor`` suffix is not used.
-  A processor must be a public service because it is loaded on demand.
-  The ``priority`` attribute is used to control the order in which processors are executed. The highest the priority, the earlier a processor is executed. Default value is 0. The possible range is from -255 to 255. But for some types of processors the range can be different. More details you can find in the `documentation of the ChainProcessor <https://github.com/oroinc/platform/tree/master/src/Oro/Component/ChainProcessor/README.md#types-of-processors>`__ component. If several processors have
   the same priority the order they are executed is unpredictable.
-  Each processor should check whether its' work is already done, because there can be a processor with higher priority which does the same but in another way. For example such processors can be created for customization purposes.
-  As Data API resources can be created for any type of objects, not only ORM entities, it is always a good idea to check whether a processor is applicable for ORM entities. This check is very fast and allows to avoid possible logic issues and performance impact. Please use the ``oro_api.doctrine_helper`` service to get an instance of
   `Oro\Bundle\ApiBundle\Util\DoctrineHelper` <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Util/DoctrineHelper.php>`__ as this class is optimized to be used in Data API stack. An example:

.. code:: php

        public function process(ContextInterface $context)
        {
            $entityClass = $context->getClassName();
            if (!$this->doctrineHelper->isManageableEntityClass($entityClass)) {
                // only manageable entities are supported
                return;
            }

            // do some work
        }

The list of all existing processors you can find in the `Processor <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Processor>`__ folder.

Processor conditions
--------------------

When you register a processor in the dependency injection container you can specify conditions when the processor should be executed. The attributes of the ``oro.api.processor`` tag is used to specify conditions. Any context property which is scalar, array or object (instance of the `ToArrayInterface <https://github.com/oroinc/platform/tree/master/src/Oro/Component/ChainProcessor/ToArrayInterface.php>`__) can be used in the conditions. Lets see a very simple condition which is used to filter
processors by the action:

.. code:: yaml

    services:
        acme.api.processor.do_something:
            class: Acme\Bundle\ProductBundle\Api\Processor\DoSomething
            tags:
                - { name: oro.api.processor, action: get }

In this case the ``acme.api.processor.do_something`` will be executed only in scope of the ``get`` action, for other actions this processor will be skipped.

The main goal of the conditions is to provide a simple way to specify which processors are required to accomplish some work. Also it is very important to understand that the processors are not fit the conditions will not be loaded from the dependency injection container at all. So, using of the conditions allows to create fast Data API.

This allows to build conditions based on any attribute from the context.

The types of conditions depend on registered `Applicable Checkers <https://github.com/oroinc/platform/tree/master/src/Oro/Component/ChainProcessor/README.md#applicable-checkers>`__. By default the following checkers are registered:

-  `MatchApplicableChecker <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Processor/MatchApplicableChecker.php>`__

Also, by performance reasons, the functionality of `SkipGroupApplicableChecker <https://github.com/oroinc/platform/tree/master/src/Oro/Component/ChainProcessor/SkipGroupApplicableChecker.php>`__ and `GroupRangeApplicableChecker <https://github.com/oroinc/platform/tree/master/src/Oro/Component/ChainProcessor/GroupRangeApplicableChecker.php>`__ was implemented as part of
`OptimizedProcessorIterator <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Processor/OptimizedProcessorIterator.php>`__.

Examples of processor conditions
--------------------------------

-  No conditions. A processor is executed for all actions.

.. code:: yaml

        tags:
            - { name: oro.api.processor }

-  A processor is executed only for a specified action.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list }

-  A processor is executed only for a specified action and group.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list, group: initialize }

-  A processor is executed only for a specified action, group and request type.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list, group: initialize, requestType: rest }

-  A processor is executed for all requests except a specified one.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list, group: initialize, requestType: !rest }

-  A processor is executed only for REST requests conform `JSON.API <http://jsonapi.org/>`__ specification.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list, group: initialize, requestType: rest&json_api }

-  A processor is executed either for REST requests or requests conform `JSON.API <http://jsonapi.org/>`__ specification.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list, group: initialize, requestType: rest|json_api }

**Please note** that a value can contain either ``&`` (logical AND) or ``|`` (logical OR) operators, but it is not possible to combine them.

-  A processor is executed for all REST requests excluding requests conform `JSON.API <http://jsonapi.org/>`__ specification.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list, group: initialize, requestType: rest&!json_api }

-  A processor is executed for several specified actions.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get, group: initialize, priority: 10 }
            - { name: oro.api.processor, action: get_list, group: initialize, priority: 5 }

-  A processor is executed only for a specified entity.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list, group: initialize, class: 'Oro\Bundle\UserBundle\Entity\User' }

-  A processor is executed only for entities that implement some interface or extend some base class. Currently there are two attributes that being compared by **instance of** instead of **equal** operator. These attributes are **class** and **parentClass**.

.. code:: yaml

        tags:
            - { name: oro.api.processor, action: get_list, group: initialize, class: 'Oro\Bundle\UserBundle\Entity\AbstractUser' }

More examples you can find in `configuration of existing processors <../config>`__. See ``processors.*.yml`` files.

Error handling
--------------

There are several types of errors that may occur during the process of a request:

-  **Validation errors**. A validation error will occur if a request has some invalid parameters, headers or data.
-  **Security errors**. This type of error will occur if an access is denied to a requested, updating or deleting entity.
-  **Unexpected errors**. These errors will occur if some unpredictable problem happens. E.g. no access to a database or a file system, requested entity does not exist, updating entity is blocked, etc.

Please note that to validate input data for `create <./actions.md#create-action>`__ and `update <./actions.md#update-action>`__ actions the best solution is to use validation constraints. In most cases it allows to not write any PHP code and configure required validation rules in ``Resources/config/oro/api.yml``. The detailed information how to add own validation constraints can be found in `Forms and Validators Configuration <./forms.md>`__ document. The following example shows how to add a
validation constraint via ``Resources/config/oro/api.yml``:

.. code:: yaml

    api:
        entities:
            Acme\Bundle\AcmeBundle\Entity\AcmeEntity:
                fields:
                    primaryEmail:
                        form_options:
                            constraints:
                                # add Symfony\Component\Validator\Constraints\Email validation constraint
                                - Email: ~

If an error occurs in a processor, the main execution flow is interrupted and the control is passed to a special group of processors, that is named **normalize\_result**. This is true for all types of errors. But there are some exceptions for this rule for the errors that occur in any processor of the **normalize\_result** group. The execution flow is interrupted only if any of these processors raises an exception. However, these processors can safely add new errors into the
`Context <./actions.md#context-class>`__ and the execution of the next processors will not be interrupted. For implementation details see `RequestActionProcessor <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Processor/RequestActionProcessor.php>`__.

An error is represented by `Error <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Model/Error.php>`__ class. Also there is `ErrorSource <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Model/ErrorSource.php>`__ class that can be used to specify a source of an error, e.g. the name of URI parameter or the path to a property in request data. These classes have the following methods:

**Error** class

-  **create(title, detail)** *static* - Creates an instance of **Error** class.
-  **createValidationError(title, detail)** *static* - Creates an instance of **Error** class represents a violation of validation constraint.
-  **createByException(exception)** *static* - Creates an instance of **Error** class based on a given exception object.
-  **getStatusCode()** - Gets the HTTP status code applicable to this problem.
-  **getCode()** - Gets an application-specific error code.
-  **setCode(code)** - Sets an application-specific error code.
-  **getTitle()** - Gets a short, human-readable summary of the problem that should not change from occurrence to occurrence of the problem.
-  **setTitle(title)** - Sets a short, human-readable summary of the problem that should not change from occurrence to occurrence of the problem.
-  **getDetail()** - Gets a human-readable explanation specific to this occurrence of the problem.
-  **setDetail(detail)** - Sets a human-readable explanation specific to this occurrence of the problem.
-  **getSource()** - Gets instance of `ErrorSource <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Model/ErrorSource.php>`__ represents a source of this occurrence of the problem.
-  **setSource(source)** - Sets instance of `ErrorSource <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Model/ErrorSource.php>`__ represents a source of this occurrence of the problem.
-  **getInnerException()** - Gets an exception object that caused this occurrence of the problem.
-  **setInnerException(exception)** - Sets an exception object that caused this occurrence of the problem.
-  **trans(translator)** - Translates all attributes that are represented by the `Label <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Model/Label.php>`__ object.

**ErrorSource** class

-  **createByPropertyPath(propertyPath)** *static* - Creates an instance of **ErrorSource** class represents the path to a property caused the error.
-  **createByPointer(pointer)** *static* - Creates an instance of **ErrorSource** class represents a pointer to a property in the request document caused the error.
-  **createByParameter(parameter)** *static* - Creates an instance of **ErrorSource** class represents URI query parameter caused the error.
-  **getPropertyPath()** - Gets the path to a property caused the error. E.g. "title", or "author.name".
-  **setPropertyPath(propertyPath)** - Sets the path to a property caused the error.
-  **getPointer()** - Gets a pointer to a property in the request document caused the error. For JSON documents the pointer conforms `RFC 6901 <https://tools.ietf.org/html/rfc6901>`__. E.g. "/data" for a primary data object, or "/data/attributes/title" for a specific attribute.
-  **setPointer(pointer)** - Sets a pointer to a property in the request document caused the error.
-  **getParameter()** - Gets URI query parameter caused the error.
-  **setParameter(parameter)** - Sets URI query parameter caused the error.

Lets consider how a processor can inform that some error is occurred.

The simplest way is just throw an exception. For example:

.. code:: php

    <?php

    namespace Oro\Bundle\ApiBundle\Processor\Shared;

    use Doctrine\ORM\QueryBuilder;

    use Oro\Component\ChainProcessor\ContextInterface;
    use Oro\Component\ChainProcessor\ProcessorInterface;
    use Oro\Component\EntitySerializer\EntitySerializer;
    use Oro\Bundle\ApiBundle\Exception\RuntimeException;
    use Oro\Bundle\ApiBundle\Processor\Context;

    /**
     * Loads entity using the EntitySerializer component.
     * As returned data is already normalized, the "normalize_data" group will be skipped.
     */
    class LoadEntityByEntitySerializer implements ProcessorInterface
    {
        /** @var EntitySerializer */
        protected $entitySerializer;

        /**
         * @param EntitySerializer $entitySerializer
         */
        public function __construct(EntitySerializer $entitySerializer)
        {
            $this->entitySerializer = $entitySerializer;
        }

        /**
         * {@inheritdoc}
         */
        public function process(ContextInterface $context)
        {
            /** @var Context $context */

            if ($context->hasResult()) {
                // data already retrieved
                return;
            }

            $query = $context->getQuery();
            if (!$query instanceof QueryBuilder) {
                // unsupported query
                return;
            }

            $config = $context->getConfig();
            if (null === $config) {
                // an entity configuration does not exist
                return;
            }

            $result = $this->entitySerializer->serialize($query, $config);
            if (empty($result)) {
                $result = null;
            } elseif (count($result) === 1) {
                $result = reset($result);
            } else {
                throw new RuntimeException('The result must have one or zero items.');
            }

            $context->setResult($result);

            // data returned by the EntitySerializer are already normalized
            $context->skipGroup('normalize_data');
        }
    }

This way is good to for unexpected and security errors (for security errors just throw ``Symfony\Component\Security\Core\Exception\AccessDeniedException``). The raised exception will be converted to the **Error** object automatically by `RequestActionProcessor <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Processor/RequestActionProcessor.php>`__. The all sensible properties of such error objects, like HTTP status code, title and description, are filed based on the
underlying exception object. This is done automatically by services that is named as exception text extractors. The default implementation of such extractor is `ExceptionTextExtractor <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Request/ExceptionTextExtractor.php>`__. To add new extractor just create a class implements
`ExceptionTextExtractorInterface <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Request/ExceptionTextExtractorInterface.php>`__ and tag it with the ``oro.api.exception_text_extractor`` in the dependency injection container.

The another way is to add an **Error** object to the context. This way is good for validation errors because it allows to add several errors. The following example demonstrate it:

.. code:: php

    <?php

    namespace Oro\Bundle\ApiBundle\Processor\Shared;

    use Oro\Component\ChainProcessor\ContextInterface;
    use Oro\Component\ChainProcessor\ProcessorInterface;
    use Oro\Bundle\ApiBundle\Model\Error;
    use Oro\Bundle\ApiBundle\Processor\SingleItemContext;
    use Oro\Bundle\ApiBundle\Request\Constraint;

    /**
     * Makes sure that the identifier of an entity exists in the Context.
     */
    class ValidateEntityIdExists implements ProcessorInterface
    {
        /**
         * {@inheritdoc}
         */
        public function process(ContextInterface $context)
        {
            /** @var SingleItemContext $context */

            $entityId = $context->getId();
            if (empty($entityId)) {
                $context->addError(
                    Error::createValidationError(
                        Constraint::ENTITY_ID,
                        'The identifier of an entity must be set in the context.'
                    )
                );
            }
        }
    }

Please note that by default the HTTP status code for validation errors is ``400 Bad Request``. But, if needed, an another HTTP status code can be set, e.g. by passing it as a third argument of the ``Error::createValidationError`` method.

Also there is the `Constraint <https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/ApiBundle/Request/Constraint.php>`__ class that contains titles for different kind of validation errors. As you can see all titles end with **constraint** word. So, while adding own types please do the same. This is not a strict rule, but it allows to keep Data API consistency.

Sometime you may need to use Data API logger directly in your processors. Actually all Data API logs are written into **api** channel. So, injecting the logger into your processor or other service can be done in a `common way <http://symfony.com/doc/current/reference/dic_tags.html#monolog-logger>`__. For example:

.. code:: yaml

        acme.api.some_processor:
            class: Acme\Bundle\AcmeBundle\Api\Processor\DoSomething
            arguments:
                - '@logger'
            tags:
                - { name: oro.api.processor, ... }
                - { name: monolog.logger, channel: api }
