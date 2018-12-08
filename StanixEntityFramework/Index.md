# StanixEntityFramework
*(This is temporary name - if you have any ideas on how to name this feel free to create issue)*

StanixEntityFramework is code-generation framework for easier handling of entity state.

## Basic atoms
1. `public struct State` - structure that stores `State` and manages it's mutations
2. `public struct StateMutation` - structure that saves information about `Mutation` of `State`
3. `public interface StateGroup` - interface that stores multiple states into logical group.
4. `[EntityModel] public interface Entity` - definition of `Entity`

## Transactions
Each state change is represented as `Transaction<StateMutation>`. Such Transactions might depend on other `State` being in exact state.

***Example*** `Transaction<HealthDamage>` might depend on `DamageResistance` having `armour` equal to 10. But another transaction could change that value resulting in recalculation of damage that is done.

This is why each Transaction has corresponding `Action` that takes some parameters and recalculates `Transaction<StateMutation>`, thus making network transactions easy to reproduce remotely.

***Implementation details*** Action is just lambda-delegate that takes some arguments and produces `Transaction<T>`

While in some cases recalculations are possible, in others ( for ex. `Transaction<InventoryTransfer>` ) rollback is required. But each Transaction might produce another Transactions, and cancelling one requires entire chain of Transactions to be cancalled - and this is where TransactionScope comes in

## TransactionScope

TransactionScope is combination of multiple Transactions and other TransactionScopes that might depend on state that is changed by this scope.

## Transaction collisions
In some cases Transaction could rollback another Transaction - and while in most cases it's something that everyone wants, but in other cases this might not be as ideal solution.

***Example***
1. User A uses some ability that makes him invulnerable
2. User B attacks User A and one-shots him
3. ???

> Like, in that case I would say that player who activated invulnerability wins .
> As it does not feel right to die after you clutched hard,
> and makes you feel like a god if you do survive that .
> But if you die ... you fell robbed and bad .
> While on the other end of killing if you one-shot someone you wouldn't even know that they activated ability!..
> ... But if they do activate ability and not die you will think that they are gods, and that they are good.
> But you wont feel robbed or bad - as they just made insane move.
> ( And they used up some resource for your action . )

If you think about UX User A shouldn't die as that makes feel him bad, but if he doesn't die everyone would be happy.

# Behind the scenes

1. We create structures that represent basic things such as Health, Identity, Inventory/Container, etc
2. We create interfaces - they are mostly for convinience and not to repeat ourself too much.
Aka we dont need to have Health in liveable - we could add that field manually to Player/NPC - but do we want that ? ;-)
3. Some of interfaces would be generated as Unity's component - we would use EntityModel annotation for that.
(T4 would handle that - it wont be able to auto regenerate everything each change but still its just 1 button click )
4. Profit ???

# Why this instead of X?

Why this and not something else:
Unity component system is kind-of messy.
Each component would need native object, and uses a lot of memory - cuz of that we wouldnt want to make each little thing as a component.
Because of that we cannot make health just as component - sadface.
And yes - Unity component system does support inheritance, but you still would need to have interfaces or complex class-based system that isn't easy to extend / reuse.

And what else, besides unity component system ? Any other examples ?
**Create issue ticket if you have any other idea**

# Example code
```
// This is definition of our model

[
public struct Health
{
   ... any code u want ...
   public void ApplyDamage(Transaction<Damage> dmg)
   ...
}

public interface Liveable
{
	Health Health { get; }
}

public interface Humanoid : Liveable
{
	...
}

[EntityModel(hooks = typeof(Hooks))]
public interface Player : Humanoid, ItemHolder, DamageDealer
{
	public class Hooks
	{
		public void Start()
		{
			// Lets hope you can guess what is this ? ;-)
		}
	}
	...
}

[EntityModel()]
public interface NPC : Liveable, ItemHolder, DamageDealer
{
}

... And somewhere in whatever you want ...
entity
	.Provide(Keys.Health)
	.ApplyDamage(dmgTransaction)
	
/* var entity = gameObject.GetComponent<Entity>() */
```

*TODO: split up into multiple documents*
*TODO: Add actuall examples after library is implemented*
*TODO: Add enumeration of pre-made Keys categories, and explain more about what `Key` is*
*TODO: Explain what Hooks/EventSystem are.*
*TODO: Actual documentation of library's code*