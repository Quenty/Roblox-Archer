# ROBLOX ARCHER


Roblox Archer is a handy utility which should rapidly speedup any "range checks" you make over time. In the core, it is a binary search tree, which allows to find nearby instances as fast as possible.

Roblox Archer is named "Archer" because it will perform "Jobs" for given "Targets". What must you do to use it?

*	Provide all instances (models or baseparts) where jobs MIGHT be performed on.
*	Provide Job objects which define what should be done for all targets.
*	Start a thread!

Note that this utility has to be used well - make sure that you understand this documentary first, because you should maximize the potential of this utility to make it significantly faster than other utilities.

# In-depth

## Jobs
Jobs are tables. 

Every :Parse field can return a number called the "weight". This will be added to the Archer.Iterations property. For heavy jobs, you can return a high weight value, which means that the thread yields more, thus freeing up space for other jobs than Archer itself. Note that high weight values will make the progress slower. Also, every check on an Instance is a Job with weight 1.

### Mandatory fields

#### Job:ShouldRun(Target)
This method should return a boolean if this job should actually be performed. This is used for a quick-checking if resources should be spent on performing this job. 

#### Job:Parse(Target, OriginalTarget, Range)

Parse the Job for the Target, from OriginalTarget. The OriginalTarget is the Target passed to the ShouldRun method. The Range is a number, which is the magnitude between the postiion vectors of Target and OriginalTarget. This function is NEVER called 
so that OriginalTarget == Target 

### Optional fields

#### Job:ParseSelfFirst(Target)

Before parsing all other items aorund the Target, call this method first.

#### Job:ParseSelfLast(Target)

After parsing all the other items around the Target, call this method.

#### Job.Range 

This is not a method, but a property. You can set this Range property to define what range this job can maximally be cast on. If this is not used, it will use Archer.Range for the Range instead (size of a instance box). If this is set to 0, :Parse() will never be called. You can use this to put functions in which you would check over time on "all instances" (for example). Instead of using another loop, you can then provide it in the Archer loop.

## Archer

### Settings 

#### Archer.Range

Defines the size of a box to store instances in. You should find a good balance in this. Note: Archer will currently only check for neighbouring boxes. This means that if you have a Job with a Range > Archer.Range, it will (in most cases) MISS certain instances. To fix this, you should make this range bigger. 

Note that if Archer.Range = a huge number a normal "job loop" is created without the advantages of the binary search. You can use this to benchmark the utility. math.huge will not work, as 0 * math.huge is undefined (check the :GetNode function, you can change this to define it for behaviour for math.huge))

#### Archer.JobTimer

This Timer is a number which defines after how many "weight" the thread should yield. Normally this is 1000. Every Instance check adds 1 to this value.

#### Archer.Mode

Archer has two modes: PerInstance and PerJob. These only apply to the Archer:StartThread() method. PerInstance will loop over all instances in the tree, and then looping over all jobs. PerJob will loop over all jobs and then over all Instances. This is normally set to PerInstance. Any other property will error on starting :StartThread().

#### [private] Archer.Iterations

The number which holds how many weight has been collected. If this is >= JobTimer, the thread will wait().

#### [private] Archer.Jobs

The list of all Jobs available to Archer.

#### [private] Archer.Tree

The BST of Archer.

### Methods

#### Archer:AddJob(Job)

Adds the Job to Archer.Jobs. It checks if the :ShouldRun and :Parse field are available. If not, this function errors.

#### Archer:Add(Instance)

Adds an Instance to Archers Tree.

#### Archer:StartThread()

Creates a new Thread which will perform the jobs in the given mode.

#### Archer:DoJob(Job)

You can also manually call Jobs (for instnace, to create a manual scheduler). You need to provide a Job instance. This function assumes that the Job is correct (if not, it will - of course - error, later on) and has all mandatory fields. If no job is provided, Archer will perform a full cycle by looping over all instances and then all jobs.

#### Archer:ParseInstance(Target, NodeIdent, Job)

This function can be manually called, but due to Archers setup there is a problem. Let's first look at the function itself. Target is the Target instance. NodeIdent is the Node identifier, which means that Archer.Tree[NodeIdentifier] is the table containing the Target. Due to how Archer creates the Data tree, this is also the ONLY table which contains this target. The problem is this NodeIdentifier. This is not stored anywhere besides the tree itself. To retrieve it:

1. Loop over the whole tree to find it. BAD, but works.
2. Try to figure out where it could be located by calling the :GetNode function.
3. Change Archer so it stores the NodeIdent in another table too. (not available as of this version yet, feel free to issue a pull request)

Job is the Job object to parse. If Job is nil, Archer will check all Jobs.


#### [private] Archer:GetNode(Position)

Return the node identifier for the given Position.

#### [private] Archer:Insert(Instance, NodeIdentifier)

Inserts the Instance inside the BST identified by the NodeIdentifier.

#### [private] Archer:ValidateNode(CurrentNode, Target)

Validates if the Target should (still) be in the Current Node CurrentNode (a Node Identifier). If not, it will fix this. There are three returned values: a boolean which holds if changes have been made, a boolean which holds if the target is completely removed from the tree and a NodeIdentifier which is the NodeIdentifier the Instance is currently in.

#### [private] Archer:GetNearbyNodes(Position, Range)

Returns a table of all nodes near Position given the Range. Only checks nearby nodes. If Range > Archer.Range, some Nodes will be left out and thus some instances which SHOULD be targeted will not get targeted. This is fixable but will make Archer slower.

#### [private] Archer:GetPosition(Instance)

Returns the Position of the given Instance, which can be a BasePart or a Model.

#### [private] Archer:CheckIter()

Checks the Iteration timer, if Archer.Iterations >= Archer.JobTimer it will reset the Iteration timer to 0 and it will wait().


