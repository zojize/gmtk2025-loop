<script lang="ts" setup>
import RAPIER from '@dimforge/rapier2d'

const WIDTH = 2000
const HEIGHT = 2000
const ZOOM = 100

const gravity = new RAPIER.Vector2(0.0, -9.81)
const world = new RAPIER.World(gravity)

// Create Ground.
const bodyDesc = RAPIER.RigidBodyDesc.fixed()
const body = world.createRigidBody(bodyDesc)
const colliderDesc = RAPIER.ColliderDesc.cuboid(10.0, 0.1)
world.createCollider(colliderDesc, body)

// Create Left Wall.
const leftWallDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(-10.0, 10.0)
const leftWall = world.createRigidBody(leftWallDesc)
const leftWallColliderDesc = RAPIER.ColliderDesc.cuboid(0.1, 10.0)
world.createCollider(leftWallColliderDesc, leftWall)

// Create Right Wall.
const rightWallDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(10.0, 10.0)
const rightWall = world.createRigidBody(rightWallDesc)
const rightWallColliderDesc = RAPIER.ColliderDesc.cuboid(0.1, 10.0)
world.createCollider(rightWallColliderDesc, rightWall)

// Create Ceiling.
const ceilingDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(0.0, 20.0)
const ceiling = world.createRigidBody(ceilingDesc)
const ceilingColliderDesc = RAPIER.ColliderDesc.cuboid(10.0, 0.1)
world.createCollider(ceilingColliderDesc, ceiling)

// Dynamic cubes.
const rad = 0.5
const num = 5
let i, k
const shift = rad * 2.5
const center = num * rad

for (i = 0; i < num; ++i) {
  for (k = i; k < num; ++k) {
    const x = (i * shift) / 2.0 + (k - i) * shift - center
    const y = (i * shift) / 2.0

    // Create dynamic cube.
    const bodyDesc = RAPIER.RigidBodyDesc.dynamic().setTranslation(x, y)
    const body = world.createRigidBody(bodyDesc)
    const colliderDesc = RAPIER.ColliderDesc.cuboid(rad, rad / 2.0)
    world.createCollider(colliderDesc, body)
  }
}

// Character.
const characterDesc = RAPIER.RigidBodyDesc.kinematicPositionBased().setTranslation(
  -8.0,
  4.0,
)
const character = world.createRigidBody(characterDesc)
const characterColliderDesc = RAPIER.ColliderDesc.capsule(0.5, 0.6)
const characterCollider = world.createCollider(
  characterColliderDesc,
  character,
)

const characterController = world.createCharacterController(0.1)
// characterController.enableAutostep(0.7, 0.3, true)
characterController.enableSnapToGround(0.7)

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
  )

  const movement = characterController.computedMovement()
  const newPos = character.translation()
  newPos.x += movement.x
  newPos.y += movement.y
  character.setNextKinematicTranslation(newPos)
}

const timeline = useTemplateRef('timeline')

// useEventListener('keydown', (event: KeyboardEvent) => {
//   if (event.key === 'ArrowLeft') {
//     event.preventDefault()
//     velocity.x = -speed
//   }
//   if (event.key === 'ArrowRight') {
//     event.preventDefault()
//     velocity.x = speed
//   }
//   if (event.key === ' ') {
//     // Only allow jumping when grounded
//     event.preventDefault()
//     if (characterController.computedGrounded()) {
//       velocity.y = jumpForce
//     }
//   }
// })

whenever(() => timeline.value?.keys.left, () => {
  velocity.x = -speed
})
whenever(() => !timeline.value?.keys.left, () => {
  velocity.x = 0.0
})

whenever(() => timeline.value?.keys.right, () => {
  velocity.x = speed
})
whenever(() => !timeline.value?.keys.right, () => {
  velocity.x = 0.0
})
whenever(() => timeline.value?.keys.up, () => {
  if (characterController.computedGrounded()) {
    velocity.y = jumpForce
  }
})

useEventListener('keyup', (event: KeyboardEvent) => {
  if (event.key === 'ArrowLeft')
    velocity.x = 0.0
  if (event.key === 'ArrowRight')
    velocity.x = 0.0
})

const obstacles = ref<{ x: number, y: number, width: number, height: number, rotation: number }[]>([])
const player = ref<{ x: number, y: number, width: number, height: number } | null>(null)

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
  world.step()

  // Render
  obstacles.value = []
  player.value = null
  world.forEachCollider((collider) => {
    if (collider.shapeType() === RAPIER.ShapeType.Cuboid) {
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
    else if (collider.shapeType() === RAPIER.ShapeType.Capsule) {
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
  <Timeline ref="timeline" :duration="2000" />
</template>
