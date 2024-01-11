How to import packages and modules in Python?

We can import both by using 

    import packages
    from package import modules
    from packages.modules import classes

We can't directly use classes with . for ex.:
    
    from packages.modules.classes   --> is incorrect



<h3> django.contrib.auth.models.py </h3>


    classes/methods defined inside this models.py file :

        class PermissionManager
        class Permission
        class GroupManager
        class Group


        class UserManager:





        class PermissionsMixin:
            """
            Add the fields and methods necessary to support the Group and Permission
            models using the ModelBackend.
            """

            is_superuser = models.BooleanField(
                _("superuser status"),
                default=False,
                help_text=_(
                    "Designates that this user has all permissions without "
                    "explicitly assigning them."
                ),
            )
            groups = models.ManyToManyField(
                Group,
                verbose_name=_("groups"),
                blank=True,
                help_text=_(
                    "The groups this user belongs to. A user will get all permissions "
                    "granted to each of their groups."
                ),
                related_name="user_set",
                related_query_name="user",
            )
            user_permissions = models.ManyToManyField(
                Permission,
                verbose_name=_("user permissions"),
                blank=True,
                help_text=_("Specific permissions for this user."),
                related_name="user_set",
                related_query_name="user",
            )

            class Meta:
                abstract = True




        class AbstractUser(AbstractBaseUser, PermissionsMixin):
            """
            An abstract base class implementing a fully featured User model with
            admin-compliant permissions.

            Username and password are required. Other fields are optional.
            """
            
            username = models.CharField(
                _("username"),
                max_length=150, unique=True,
                help_text=_(
                    "Required. 150 characters or fewer. Letters, digits and @/./+/-/_ only."
                ),
                validators=[username_validator],
                error_messages={
                    "unique": _("A user with that username already exists."),
                },
            )

            first_name = models.CharField(_("first name"), max_length=150, blank=True)
            last_name = models.CharField(_("last name"), max_length=150, blank=True)
            email = models.EmailField(_("email address"), blank=True)

            is_staff = models.BooleanField(
                _("staff status"),
                default=False,
                help_text=_("Designates whether the user can log into this admin site."),
            )
            is_active = models.BooleanField(
                _("active"),
                default=True,
                help_text=_(
                    "Designates whether this user should be treated as active. "
                    "Unselect this instead of deleting accounts."
                ),
            )
            date_joined = models.DateTimeField(_("date joined"), default=timezone.now)

            objects = UserManager()

            EMAIL_FIELD = "email"
            USERNAME_FIELD = "username"
            REQUIRED_FIELDS = ["email"]

            class Meta:
                verbose_name = _("user")
                verbose_name_plural = _("users")
                abstract = True

            
            def get_full_name(self):
                """
                Return the first_name plus the last_name, with a space in between.
                """
                full_name = "%s %s" % (self.first_name, self.last_name)
                return full_name.strip()
            
            def get_short_name(self):
                """Return the short name for the user."""
                return self.first_name

            def email_user(self, subject, message, from_email=None, **kwargs):
                """Send an email to this user."""
                send_mail(subject, message, from_email, [self.email], **kwargs)
            
        

        class User(AbstractUser):
            """
            Users within the Django authentication system are represented by this
            model.

            Username and password are required. Other fields are optional.
            """

            class Meta(AbstractUser.Meta):
                swappable = "AUTH_USER_MODEL"



        class AnonymousUser:





<h3> django.contrib.auth.base_user.py </h3>


We can import these classes in 2 ways:


because both these classes
    
    class AbstractUser(AbstractBaseUser, PermissionsMixin) class UserManager(BaseUserManager) 

are inheriting BaseUserManager and AbstractBaseUser

    from django.contrib.auth.models import BaseUserManager, AbstractBaseUser

This is the main module where both these classes are defined 

    from django.contrib.auth.base_user import BaseUserManager, AbstractBaseUser


    class BaseUserManager



    class AbstractBaseUser
        password = models.CharField(_("password"), max_length=128)
        last_login = models.DateTimeField(_("last login"), blank=True, null=True)

        is_active = True

        # Stores the raw password if set_password() is called 
        _password = None

        @property
        def is_anonymous(self):
            """
            Always return False. This is a way of comparing User objects to anonymous users.
            """
            return False
        
        @property
        def is_authenticated(self):
            """
            Always return True. This is a way to tell if the user has been authenticated in templates.
            """
            return True

        def set_password(self, raw_password):
            self.password = make_password(raw_password)
            self._password = raw_password

        def check_password(self, raw_password):
            """
            Return a boolean of whether the raw_password was correct. Handles
            hashing formats behind the scenes.
            """
        




# what is the difference between AbstractUser and AbstracBaseUser

In Django, when we work with models for user authentication, we may encounter different base classes for creating custom user models.


## AbstractUser:

Overview:

- AbstractUser is a high-level abstract base class provided by Django for creating custom user models.
- It is an extension of the built-in User model, adding additional fields and methods commonly used for user authentication.
- This class includes fields such as username, email, password, and more, as well as methods like get_full_name and get_short_name.


Usage:

- When you inherit from AbstractUser, you get a user model with a set of common fields and methods, making it suitable for most applications.
- You can customize the model by adding your own fields or methods

Example:

    from django.contrib.auth.models import AbstractUser

    class CustomUser(AbstractUser):
        # Add custom fields if needed
        birth_date = models.DateField(null=True, blank=True)



## AbstractBaseUser:

Overview:

- AbstractBaseUser is a lower-level abstract base class provided by Django for creating custom user models with a higher level of customization.
- Unlike AbstractUser, it does not provide predefined fields like username or email. Instead, you need to define your own fields for username, password, and other user-specific attributes.
- This class gives you more control but requires you to implement more methods and fields manually.

Usage:

- Use AbstractBaseUser when you need a high level of customization for your user model, and you want to define your own fields and authentication methods.

Example:

    from django.contrib.auth.models import AbstractBaseUser, BaseUserManager

    class CustomUserManager(BaseUserManager):
        # Custom manager methods

    class CustomUser(AbstractBaseUser):
        username = models.CharField(max_length=30, unique=True)
        email = models.EmailField(unique=True)
        # Add custom fields as needed

        objects = CustomUserManager()

        # Required methods for authentication
        def has_perm(self, perm, obj=None):
            return True

        def has_module_perms(self, app_label):
            return True


## Key Differences:

- Fields and Methods:

- AbstractUser comes with predefined fields and methods commonly used in user models, while AbstractBaseUser provides minimal functionality, and you need to define your own fields and methods.


- Level of Customization:

- AbstractUser is more opinionated and convenient for typical use cases, whereas AbstractBaseUser offers greater flexibility and control, allowing you to define your user model from scratch.


- Ease of Use:

- AbstractUser is generally easier to use and suitable for most projects that don't require extensive customization.
- AbstractBaseUser is recommended for projects with specific requirements for user model structure and behavior.


In summary, choose AbstractUser for a more straightforward approach, and choose AbstractBaseUser for a higher level of customization and control over the user model structure.





# What are the different model inheritances in django

