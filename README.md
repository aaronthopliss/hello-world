# hello-world
another test
Hello my name is aaron and i am testing out git which is a good program when i can get it to work.
ps. its friday!!!!


using System.Collections.Generic;
using System.IO;
using NGit;
using NGit.Api;
using NGit.Api.Errors;
using NGit.Internal;
using NGit.Revwalk;
using Sharpen;

namespace NGit.Api
{
	/// <summary>Used to delete one or several branches.</summary>
	/// <remarks>
	/// Used to delete one or several branches.
	/// The result of
	/// <see cref="Call()">Call()</see>
	/// is a list with the (full) names of the deleted
	/// branches.
	/// Note that we don't have a setter corresponding to the -r option; remote
	/// tracking branches are simply deleted just like local branches.
	/// </remarks>
	/// <seealso><a
	/// *      href="http://www.kernel.org/pub/software/scm/git/docs/git-branch.html"
	/// *      >Git documentation about Branch</a></seealso>
	public class DeleteBranchCommand : GitCommand<IList<string>>
	{
		private readonly ICollection<string> branchNames = new HashSet<string>();

		private bool force;

		/// <param name="repo"></param>
		protected internal DeleteBranchCommand(Repository repo) : base(repo)
		{
		}

		/// <exception cref="NGit.Api.Errors.NotMergedException">
		/// when trying to delete a branch which has not been merged into
		/// the currently checked out branch without force
		/// </exception>
		/// <exception cref="NGit.Api.Errors.CannotDeleteCurrentBranchException">NGit.Api.Errors.CannotDeleteCurrentBranchException
		/// 	</exception>
		/// <returns>the list with the (full) names of the deleted branches</returns>
		/// <exception cref="NGit.Api.Errors.GitAPIException"></exception>
		public override IList<string> Call()
		{
			CheckCallable();
			IList<string> result = new AList<string>();
			if (branchNames.IsEmpty())
			{
				return result;
			}
			try
			{
				string currentBranch = repo.GetFullBranch();
				if (!force)
				{
					// check if the branches to be deleted
					// are all merged into the current branch
					RevWalk walk = new RevWalk(repo);
					RevCommit tip = walk.ParseCommit(repo.Resolve(Constants.HEAD));
					foreach (string branchName in branchNames)
					{
						if (branchName == null)
						{
							continue;
						}
						Ref currentRef = repo.GetRef(branchName);
						if (currentRef == null)
						{
							continue;
						}
						RevCommit @base = walk.ParseCommit(repo.Resolve(branchName));
						if (!walk.IsMergedInto(@base, tip))
						{
							throw new NotMergedException();
						}
					}
				}
				SetCallable(false);
				foreach (string branchName_1 in branchNames)
				{
					if (branchName_1 == null)
					{
						continue;
					}
					Ref currentRef = repo.GetRef(branchName_1);
					if (currentRef == null)
					{
						continue;
					}
					string fullName = currentRef.GetName();
					if (fullName.Equals(currentBranch))
					{
						throw new CannotDeleteCurrentBranchException(MessageFormat.Format(JGitText.Get().
							cannotDeleteCheckedOutBranch, branchName_1));
					}
					RefUpdate update = repo.UpdateRef(fullName);
					update.SetRefLogMessage("branch deleted", false);
					update.SetForceUpdate(true);
					RefUpdate.Result deleteResult = update.Delete();
					bool ok = true;
					switch (deleteResult)
					{
						case RefUpdate.Result.IO_FAILURE:
						case RefUpdate.Result.LOCK_FAILURE:
						case RefUpdate.Result.REJECTED:
						{
							ok = false;
							break;
						}

						default:
						{
							break;
							break;
						}
					}
					if (ok)
					{
						result.AddItem(fullName);
						if (fullName.StartsWith(Constants.R_HEADS))
						{
							string shortenedName = Sharpen.Runtime.Substring(fullName, Constants.R_HEADS.Length
								);
							// remove upstream configuration if any
							StoredConfig cfg = repo.GetConfig();
							cfg.UnsetSection(ConfigConstants.CONFIG_BRANCH_SECTION, shortenedName);
							cfg.Save();
						}
					}
					else
					{
						throw new JGitInternalException(MessageFormat.Format(JGitText.Get().deleteBranchUnexpectedResult
							, deleteResult.ToString()));
					}
				}
				return result;
			}
			catch (IOException ioe)
			{
				throw new JGitInternalException(ioe.Message, ioe);
			}
		}

		/// <param name="branchnames">
		/// the names of the branches to delete; if not set, this will do
		/// nothing; invalid branch names will simply be ignored
		/// </param>
		/// <returns>this instance</returns>
		public virtual NGit.Api.DeleteBranchCommand SetBranchNames(params string[] branchnames
			)
		{
			CheckCallable();
			this.branchNames.Clear();
			foreach (string branch in branchnames)
			{
				this.branchNames.AddItem(branch);
			}
			return this;
		}

		/// <param name="force">
		/// <code>true</code> corresponds to the -D option,
		/// <code>false</code> to the -d option (default) <br />
		/// if <code>false</code> a check will be performed whether the
		/// branch to be deleted is already merged into the current branch
		/// and deletion will be refused in this case
		/// </param>
		/// <returns>this instance</returns>
		public virtual NGit.Api.DeleteBranchCommand SetForce(bool force)
		{
			CheckCallable();
			this.force = force;
			return this;
		}
	}
}
