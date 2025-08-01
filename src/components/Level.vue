<script lang="ts" setup>
import type Timeline from './Timeline.vue'
import RAPIER from '@dimforge/rapier2d'

const props = withDefaults(
  defineProps<{
    characterStartPos: RAPIER.Vector
    obstacles?: { x: number, y: number, width: number, height: number }[]
    goalPos: RAPIER.Vector
    timeline: InstanceType<typeof Timeline> | null
  }>(),
  {
    obstacles: () => [],
  },
)

const emit = defineEmits<{
  goal: []
}>()

const WIDTH = 2000
const HEIGHT = 2000
const ZOOM = 100

const gravity = new RAPIER.Vector2(0.0, -9.81)
const world = new RAPIER.World(gravity)

const obstacleColliders = ref(new WeakSet<RAPIER.Collider>())
// Create Ground.
const bodyDesc = RAPIER.RigidBodyDesc.fixed()
const body = world.createRigidBody(bodyDesc)
const colliderDesc = RAPIER.ColliderDesc.cuboid(10.0, 0.1)
obstacleColliders.value.add(world.createCollider(colliderDesc, body))

// Create Left Wall.
const leftWallDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(-10.0, 10.0)
const leftWall = world.createRigidBody(leftWallDesc)
const leftWallColliderDesc = RAPIER.ColliderDesc.cuboid(0.1, 10.0)
obstacleColliders.value.add(world.createCollider(leftWallColliderDesc, leftWall))

// Create Right Wall.
const rightWallDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(10.0, 10.0)
const rightWall = world.createRigidBody(rightWallDesc)
const rightWallColliderDesc = RAPIER.ColliderDesc.cuboid(0.1, 10.0)
obstacleColliders.value.add(world.createCollider(rightWallColliderDesc, rightWall))

// Create Ceiling.
const ceilingDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(0.0, 20.0)
const ceiling = world.createRigidBody(ceilingDesc)
const ceilingColliderDesc = RAPIER.ColliderDesc.cuboid(10.0, 0.1)
obstacleColliders.value.add(world.createCollider(ceilingColliderDesc, ceiling))

for (const obstacle of props.obstacles) {
  const obstacleDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(obstacle.x, obstacle.y)
  const obstacleBody = world.createRigidBody(obstacleDesc)
  const obstacleColliderDesc = RAPIER.ColliderDesc.cuboid(obstacle.width, obstacle.height)
  obstacleColliders.value.add(world.createCollider(obstacleColliderDesc, obstacleBody))
}

// Character
const characterDesc = RAPIER.RigidBodyDesc
  .kinematicPositionBased()
  .setTranslation(props.characterStartPos.x, props.characterStartPos.y)
const character = world.createRigidBody(characterDesc)
const characterColliderDesc = RAPIER.ColliderDesc.capsule(0.5, 0.6)
const characterCollider = world.createCollider(characterColliderDesc, character)
// Enable collision events for the character
characterCollider.setActiveEvents(RAPIER.ActiveEvents.COLLISION_EVENTS)
// Set active collision types to detect fixed colliders (including sensors)
characterCollider.setActiveCollisionTypes(RAPIER.ActiveCollisionTypes.KINEMATIC_FIXED)
const characterController = world.createCharacterController(0.1)
// characterController.enableAutostep(0.7, 0.3, true)
characterController.enableSnapToGround(0.7)

// Goal
let goalDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(props.goalPos.x, props.goalPos.y)
let goal = world.createRigidBody(goalDesc)
let goalColliderDesc = RAPIER.ColliderDesc.cuboid(0.75, 0.75)
let goalCollider = world.createCollider(goalColliderDesc, goal)
goalCollider.setSensor(true)
// Enable collision events for the goal
goalCollider.setActiveEvents(RAPIER.ActiveEvents.COLLISION_EVENTS)
// Set active collision types to detect kinematic colliders
goalCollider.setActiveCollisionTypes(RAPIER.ActiveCollisionTypes.KINEMATIC_FIXED)

// Event queue for collision detection
const eventQueue = new RAPIER.EventQueue(true)

const speed = 0.15
const jumpForce = 0.4
const gravityForce = 0.015
const velocity = { x: 0.0, y: 0.0 }

function updateCharacter() {
  // Apply gravity when not grounded
  if (!characterController.computedGrounded()) {
    velocity.y -= gravityForce
  }
  else {
    // Reset vertical velocity when grounded
    if (velocity.y < 0) {
      velocity.y = 0
    }
  }

  const movementDirection = { x: velocity.x, y: velocity.y }

  characterController.computeColliderMovement(
    characterCollider,
    movementDirection,
    RAPIER.QueryFilterFlags.EXCLUDE_SENSORS,
  )

  const movement = characterController.computedMovement()
  const newPos = character.translation()
  newPos.x += movement.x
  newPos.y += movement.y
  character.setNextKinematicTranslation(newPos)
}

until(() => props.timeline)
  .toBeTruthy()
  .then((timeline) => {
    whenever(() => timeline.keys.left, () => {
      velocity.x = -speed
    })
    whenever(() => !timeline.keys.left, () => {
      if (velocity.x < 0)
        velocity.x = 0.0
    })

    whenever(() => timeline.keys.right, () => {
      velocity.x = speed
    })
    whenever(() => !timeline.keys.right, () => {
      if (velocity.x > 0)
        velocity.x = 0.0
    })
    whenever(() => timeline.keys.up, () => {
      if (characterController.computedGrounded()) {
        velocity.y = jumpForce
      }
    })
  })

const obstacles = ref<{ x: number, y: number, width: number, height: number, rotation: number }[]>([])
const player = ref<{ x: number, y: number, width: number, height: number } | null>(null)
const goalRef = ref<{ x: number, y: number, radius: number } | null>(null)

const TARGET_FPS = 144
const TARGET_FRAME_TIME = 1000 / TARGET_FPS
const FIXED_TIMESTEP = 1.0 / TARGET_FPS
let lastFrameTime = 0
const nextFrameDelay = ref(TARGET_FRAME_TIME)

const { start: scheduleNextFrame } = useTimeoutFn(() => {
  gameLoop()
}, nextFrameDelay, { immediate: false })

function gameLoop() {
  const currentTime = performance.now()

  if (lastFrameTime === 0) {
    lastFrameTime = currentTime
  }

  const deltaTime = currentTime - lastFrameTime
  lastFrameTime = currentTime

  // Set fixed timestep for consistent physics
  world.timestep = FIXED_TIMESTEP

  // Update physics
  updateCharacter()
  world.step(eventQueue)

  eventQueue.drainCollisionEvents((handle1, handle2, started) => {
    const collider1 = world.getCollider(handle1)
    const collider2 = world.getCollider(handle2)

    if (started && (
      (collider1 === characterCollider && collider2 === goalCollider)
      || (collider1 === goalCollider && collider2 === characterCollider)
    )) {
      world.removeCollider(goalCollider, true)
      emit('goal')
    }
  })

  // Render
  obstacles.value = []
  player.value = null
  goalRef.value = null
  world.forEachCollider((collider) => {
    if (obstacleColliders.value.has(collider)) {
      const shape = collider.shape as RAPIER.Cuboid
      const width = shape.halfExtents.x * 2
      const height = shape.halfExtents.y * 2

      const position = collider.translation()
      const rotation = collider.rotation() // Get rotation in radians

      // Calculate center position in screen coordinates
      const centerX = position.x * ZOOM + WIDTH / 2
      const centerY = -position.y * ZOOM + HEIGHT

      obstacles.value.push({
        x: centerX - (width * ZOOM) / 2,
        y: centerY - (height * ZOOM) / 2,
        width: width * ZOOM,
        height: height * ZOOM,
        rotation: rotation * (180 / Math.PI), // Convert to degrees for SVG
      })
    }
    else if (collider === characterCollider) {
      // Render player as capsule
      const shape = collider.shape as RAPIER.Capsule
      const width = shape.radius * 2
      const height = (shape.halfHeight * 2) + (shape.radius * 2)

      const position = collider.translation()

      // Calculate center position in screen coordinates
      const centerX = position.x * ZOOM + WIDTH / 2
      const centerY = -position.y * ZOOM + HEIGHT

      player.value = {
        x: centerX - (width * ZOOM) / 2,
        y: centerY - (height * ZOOM) / 2,
        width: width * ZOOM,
        height: height * ZOOM,
      }
    }
    else if (collider === goalCollider) {
      // Render goal as green ball
      const shape = collider.shape as RAPIER.Cuboid
      const radius = shape.halfExtents.x // Use half extent as radius

      const position = collider.translation()

      // Calculate center position in screen coordinates
      const centerX = position.x * ZOOM + WIDTH / 2
      const centerY = -position.y * ZOOM + HEIGHT

      goalRef.value = {
        x: centerX,
        y: centerY,
        radius: radius * ZOOM,
      }
    }
  })

  // Calculate next frame timing with delta time compensation
  const frameOverrun = deltaTime - TARGET_FRAME_TIME
  nextFrameDelay.value = Math.max(1, TARGET_FRAME_TIME - frameOverrun)

  scheduleNextFrame()
}
function initializeGame() {
  world.timestep = FIXED_TIMESTEP
  scheduleNextFrame()
}

initializeGame()

function restart() {
  // Reset velocity
  velocity.x = 0.0
  velocity.y = 0.0

  character.setTranslation(props.characterStartPos, true)

  if (!world.getCollider(goalCollider.handle)) {
    goalDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(props.goalPos.x, props.goalPos.y)
    goal = world.createRigidBody(goalDesc)
    goalColliderDesc = RAPIER.ColliderDesc.cuboid(0.75, 0.75)
    goalCollider = world.createCollider(goalColliderDesc, goal)
    goalCollider.setSensor(true)
    goalCollider.setActiveEvents(RAPIER.ActiveEvents.COLLISION_EVENTS)
    goalCollider.setActiveCollisionTypes(RAPIER.ActiveCollisionTypes.KINEMATIC_FIXED)
  }

  // Reset frame timing
  lastFrameTime = 0
}

defineExpose({
  restart,
})
</script>

<template>
  <svg
    xmlns="http://www.w3.org/2000/svg"
    :viewBox="`0 0 ${WIDTH} ${HEIGHT}`"
  >
    <!-- Cuboid colliders (walls, ground, ceiling, cubes) -->
    <rect
      v-for="(rect, index) in obstacles"
      :key="index"
      :x="rect.x"
      :y="rect.y"
      :width="rect.width"
      :height="rect.height"
      :transform="`rotate(${rect.rotation} ${rect.x + rect.width / 2} ${rect.y + rect.height / 2})`"
      fill="gray"
    />

    <!-- Goal as green ball -->
    <circle
      v-if="goalRef"
      :cx="goalRef.x"
      :cy="goalRef.y"
      :r="goalRef.radius"
      fill="limegreen"
    />

    <!-- Player character as blue capsule -->
    <rect
      v-if="player"
      :x="player.x"
      :y="player.y"
      :width="player.width"
      :height="player.height"
      :rx="player.width / 2"
      :ry="player.width / 2"
      fill="skyblue"
    />

  </svg>
</template>
