# Artem Danich
## Unity Game Developer
A game developer with more than 6 years of experience in the industry. Proficient in Unity Engine and C#, focused on mobile platforms. Have a wide range of soft-skills including art and animation, project management and game design. Involved in all development phases from concepts to release and maintenance. This page is designed to share the projects I worked on.
<br>
# Projects
## 1. Papaya Solitaire (Solitaire Cash)
[Appstore link](https://apps.apple.com/il/app/papaya-solitaire/id1446254576)

A flagship project of [Papaya](https://papaya.com) which is a top 1 casino game in US with over 500k daily active users.
As a part of Solitaire team I performed a massive refactoring to improve maintainability and stability of the project. 
I was heavily involved in features architecure design, worked on several features, which succesfully reached their goal KPI's and led tech aspects of the the in-game reskinning during the UK rebranding compaign.

https://github.com/user-attachments/assets/48cbb317-3dc7-4dc7-86a0-d1f7c1632e4b

## 2. Papaya Bubble (Bubble Cash)
[Appstore link](https://apps.apple.com/il/app/papaya-bubble/id1475514684)

Skill-based real money project of [Papaya](https://papaya.com). At the begninng of the journey at Papaya I worked on this project also, performing some refactorings, planning architecture design for the features and performing bug fixes. Also worked on level editor tool with other team memebers.

https://github.com/user-attachments/assets/f96bd5a0-730c-4a12-b28f-f2b102c4a904

## 3. Sew 3D
[Appstore link](https://apps.apple.com/us/app/sew-3d/id1614461317)

The game was developed under the [Friends Games](https://www.linkedin.com/company/friends-games-incubator/) and published by [Voodoo](https://voodoo.io)
During the development of this fun and relaxing game I went through the whole process of scaling the game from rough prototype to the published project. 
As a part of the team I developed most of the meta game mechanics and UI interactions, performed a lot of optimizations, managed ab tests and live operations and took care of project architecture. 
Took significant part in decision making and management of the development process.

https://github.com/user-attachments/assets/6be621ca-73ec-4d81-863b-6ca5dbde0471

## 4. Draw The Line 3D
[Appstore link](https://apps.apple.com/us/app/draw-the-line-3d/id1541146937)

The action drawing game developed with [Friends Games](https://www.linkedin.com/company/friends-games-incubator/) team and published by [Supersonic](https://supersonic.com).
I contributed to scaling the prototype to a published game, took care of UI and meta game, skins, ads, analytics, ab tests, optimization and so on.
Also took one of the key contributions in decision making and management of the game.

https://github.com/user-attachments/assets/c1b2f726-1941-4544-a028-4f183cfde008

## 5. Airport 737 Idle
[Appstore link](https://apps.apple.com/us/app/airport-737-idle/id1479743552)

The idle tycoon game self published by [Friends Games](https://www.linkedin.com/company/friends-games-incubator/). It didn't get deserved success unfortunately but it was fun to work on.
The game was developed using Unity ECS framework which helped to achieve good performance on mobiles. I contributed to develpment of several features, adding effects and UI interactions and optimizing. Also I was involved in experiments of moving the project from 2d to 3d. It was exciting but never reached the store.

https://github.com/user-attachments/assets/9a4354b2-00d5-468c-87c8-0fb40a72b6ee

# More core mechanic examples from different projects I worked on:

https://github.com/user-attachments/assets/61b963ca-d2c1-4366-be04-340f81584529

https://github.com/user-attachments/assets/09cf6f1e-df53-494a-a650-854094555a09

https://github.com/user-attachments/assets/1d4f58f9-09bc-45d7-9794-b47d4e22d8c7

https://github.com/user-attachments/assets/a55e69f7-b2d8-4823-9009-964c8ba3f80e

Most of these game prototypes can be found on [Volodymyr Delinskyi](https://apps.apple.com/us/developer/volodymyr-delinskyi/id1472993251) account that is used for Friends Games live tests.

## Code example
```
#if UNITASK
using System;
using System.Collections.Generic;
using System.Threading;
using Cysharp.Threading.Tasks;

namespace GothmogToolkit.Tools.Core.StatesHandler
{
	public class StatesHandlerMachine
	{
		private readonly Dictionary<Type, IState> _states = new(8);

		public event Action<IState> EnteringState;
		public event Action<IState> ExitedState;

		public void Initialize(IEnumerable<IState> states)
		{
			foreach (var state in states)
			{
				RegisterState(state);
			}
		}

		private void RegisterState(IState state)
		{
			if (state == null)
				throw new ArgumentNullException($"Failed to register state. The state is null.");
			
			if (state.Type == null)
				throw new ArgumentNullException($"Failed to register state. Type of the state {state} is null.");
			
			if (!_states.TryAdd(state.Type, state))
				throw new ArgumentException(
					$"Failed to register state. State of type {state.Type} is already registered");
		}

		public async UniTask Run<TFirstState>(CancellationToken cancellationToken, bool shouldYieldBetweenStates = false)
			where TFirstState : State
		{
			var nextState = typeof(TFirstState);
			StateTransition lastTransition = null;

			while (nextState != null)
			{
				if (!TryGetState(nextState, out var state))
					throw new ArgumentException($"No state of type {nextState} found");

				try
				{
					cancellationToken.ThrowIfCancellationRequested();
					
					EnteringState?.Invoke(state);
					lastTransition = await state.Execute(cancellationToken, lastTransition?.TransitionArgs);
					ExitedState?.Invoke(state);
				}
				catch (OperationCanceledException)
				{
					return;
				}

				nextState = lastTransition.NextStateType;
				if (lastTransition?.NextStateType == null)
					return;
				
				if(shouldYieldBetweenStates)
					await UniTask.Yield();
			}
		}

		public bool TryGetState(Type type, out IState state) => _states.TryGetValue(type, out state);
	}
}
#endif
```

