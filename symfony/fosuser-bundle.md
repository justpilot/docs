# FOSUserBundle Integration with Symfony

> [!WARNING]
> The use of FOSUserBundle is no longer recommended for better flexibility and security. It is advised to switch to the
> [Doctrine entity user provider](https://symfony.com/doc/current/security/user_provider.html#entity-user-provider) (recommended)
> or consider [creating a custom user provider](https://symfony.com/doc/current/security/user_provider.html#creating-a-custom-user-provider).

## Installing the Bundle

The installation procedure of the FOSUserBundle is described [in the FOSUserBundle documentation](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/docs/index.rst).

You can:

- Skip [step 3 (Create your User class)](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/docs/index.rst#step-3-create-your-user-class)
  and use the class provided in the next paragraph to set up serialization groups the correct way
- Skip [step 4 (Configure your application's security.yml)](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/docs/index.rst#step-4-configure-your-applications-securityyml)
  if you are planning to [use a JWT-based authentication using `LexikJWTAuthenticationBundle`](../core/jwt.md)

If you are using the API Platform Standard Edition, you will need to enable the form services in the symfony framework
configuration options:

```yaml
# api/config/packages/framework.yaml
framework:
  form: { enabled: true }
```

## Creating a `User` Entity with Serialization Groups

Here's an example of declaration of a [Doctrine ORM User class](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/docs/index.rst#a-doctrine-orm-user-class).
There's also an example for a [Doctrine MongoDB ODM](https://github.com/FriendsOfSymfony/FOSUserBundle/blob/master/docs/index.rst#b-mongodb-user-class).
You need to use serialization groups to hide some properties like `plainPassword` (only in read) and `password`. The properties
shown are handled with [`normalizationContext`](../core/serialization.md#normalization), while the properties
you can modify are handled with [`denormalizationContext`](../core/serialization.md#denormalization).

Create your User entity with serialization groups:

```php
<?php
// api/src/Entity/User.php
namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use Doctrine\ORM\Mapping as ORM;
use FOS\UserBundle\Model\User as BaseUser;
use FOS\UserBundle\Model\UserInterface;
use Symfony\Component\Serializer\Annotation\Groups;

#[ORM\Entity]
#[ORM\Table(name: 'fos_user')]
#[ApiResource(
    normalizationContext: ['groups' => ['user']],
    denormalizationContext: ['groups' => ['user', 'user:write']],
)]
class User extends BaseUser
{
    #[ORM\Id, ORM\Column, ORM\GeneratedValue]
    protected ?int $id = null;

    #[Groups("user")]
    protected string $email;

    #[ORM\Column(nullable: true)]
    #[Groups("user")]
    protected string $fullname;

    #[Groups("user:write")]
    protected string $plainPassword;

    #[Groups("user")]
    protected string $username;

    public function setFullname(?string $fullname): void
    {
        $this->fullname = $fullname;
    }

    public function getFullname(): ?string
    {
        return $this->fullname;
    }

    public function isUser(?UserInterface $user = null): bool
    {
        return $user instanceof self && $user->id === $this->id;
    }
}
```
