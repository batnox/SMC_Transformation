//------------------------------------------------------------------------------
// AST matching sample. Demonstrates:
//
// * How to write a simple source tool using libTooling.
// * How to use AST matchers to find interesting AST nodes.
// * How to use the Rewriter API to rewrite the source code.
//
// Eli Bendersky (eliben@gmail.com)
// This code is in the public domain
//------------------------------------------------------------------------------
#include <string>
#include <iostream>
#include "clang/AST/AST.h"
#include "clang/AST/ASTConsumer.h"
#include "clang/ASTMatchers/ASTMatchFinder.h"
#include "clang/ASTMatchers/ASTMatchers.h"
#include "clang/Frontend/CompilerInstance.h"
#include "clang/Frontend/FrontendActions.h"
#include "clang/Rewrite/Core/Rewriter.h"
#include "clang/Tooling/CommonOptionsParser.h"
#include "clang/Tooling/Tooling.h"
#include "llvm/Support/raw_ostream.h"
#include "clang/AST/Type.h"
#include "llvm/ADT/StringRef.h"
#include "clang/AST/ExprCXX.h"
#include "clang/Lex/Lexer.h"
#include "clang/AST/Expr.h"
#include "clang/AST/Decl.h"
#include "clang/Lex/PPCallbacks.h"
#include "clang/Basic/Module.h"
#include "clang/Lex/Preprocessor.h"
#include "clang/Basic/FileManager.h"


using namespace clang;
using namespace clang::ast_matchers;
using namespace clang::driver;
using namespace clang::tooling;
bool cudafunction = false;
bool firststmt = false;
unsigned int line_num;
static llvm::cl::OptionCategory MatcherSampleCategory("Matcher Sample");

//class HeaderFileHandler : public MatchFinder::MatchCallback{
//	public: 
//		HeaderFileHandler(Rewriter &Rewrite) : Rewrite(Rewrite){}
//		
//		virtual void run(const MatchFinder::MatchResult &Result){
//			const Stmt *first = Result.Nodes.getNodeAs<clang::Stmt>("statement");
//			if(!firststmt){
//				Rewrite.InsertText(first->getLocStart(),"LALALALALALALALALALALLALAL\n", true, true);
//				//firststmt = true;
//			}
//		}
//	private:
//		Rewriter &Rewrite;
//};
class IncludeHandler : public PPCallbacks
{
	public:
		IncludeHandler(Rewriter &Rewrite) : Rewrite(Rewrite) {}  	
         	    void InclusionDirective( SourceLocation source_loc, const Token &include_smc, StringRef file_name, bool angled_bracket, CharSourceRange file_range, const FileEntry *file, StringRef search_path, StringRef relative_path, const Module *imported)
			      {	      	      
				      if (file_name.find("helper_cuda.h") != std::string::npos){

				      SourceLocation source_location = source_loc.getLocWithOffset(24);
				      Rewrite.InsertText(source_location,"\n#include \"smc.h\" \n ", true,true);	
				      }

			      }
private:
	Rewriter &Rewrite;
 };

//class IncludeHandler : public PPCallbacks
//{
//public:
//	bool has_include = false;
//	IncludeHandler(const clang::CompilerInstance &compiler) : compiler(compiler) {}
//	void InclusionDirective(SourceLocation source_loc, const Token &include_smc, StringRef file_name, bool angled_bracket, CharSourceRange file_range, const FileEntry *file, StringRef search_path, StringRef relative_path, const clang::Module *imported){
//	clang::SourceManager &source_manager = compiler.getSourceManager();
//	if(source_manager.isInMainFile(source_loc)){
//		has_include = true;
//		const unsigned int line_number = source_manager.getSpellingLineNumber(source_loc);
//		line_num = line_number;
//	}
//
//}
//private:
//	const clang::CompilerInstance &compiler;
//};

class CUDACallHandler : public MatchFinder::MatchCallback{
	public:
		CUDACallHandler(Rewriter &Rewrite) : Rewrite(Rewrite){}

		virtual void run(const MatchFinder::MatchResult &Result){
			const CUDAKernelCallExpr *cu = Result.Nodes.getNodeAs<clang::CUDAKernelCallExpr>("cudaCall");
			Rewrite.InsertText(cu->getLocEnd()/*.getLocWithOffset(-1)*/, ", __SMC_orgGridDim, __SMC_workersNeeded, __SMC_workerCount, __SMC_newChunkSeq, __SMC_seqEnds", true, true);
			Rewrite.InsertText(cu->getLocStart(), "__SMC_init()\n", true, true);

		}
	private:
		Rewriter &Rewrite;

};

class VarDeclHandler : public MatchFinder::MatchCallback{
	public:
		VarDeclHandler(Rewriter &Rewrite) : Rewrite(Rewrite){}

		virtual void run(const MatchFinder::MatchResult &Result) {
			const VarDecl *v = Result.Nodes.getNodeAs<clang::VarDecl>("varDecl");
			if(v->getNameAsString() == "grid"){
				const std::string grid_string = "dim3 __SMC_orgGridDim";
				StringRef grid_sref = StringRef(grid_string);
				Rewrite.ReplaceText(v->getLocStart(),9, grid_sref);
				
			}
		}
	private:
		  Rewriter &Rewrite;

};

class FunctionHandler : public MatchFinder::MatchCallback {
public:
  FunctionHandler(Rewriter &Rewrite) : Rewrite(Rewrite) {}

  virtual void run(const MatchFinder::MatchResult &Result) {
    // The matched 'if' statement was bound to 'ifStmt'.
      	const FunctionDecl *f = Result.Nodes.getNodeAs<clang::FunctionDecl>("functionDecl");
	//DeclarationName DeclName = f->getNameInfo().getName();
	//std::string FuncName = DeclName.getAsString();
	//std::cout<<FuncName<<"\n";

	if(f->hasAttr<CUDAGlobalAttr>()){
	//if(true){
		//std::cout<<"hi\n";
		DeclarationName DeclName = f->getNameInfo().getName();
		std::string FuncName = DeclName.getAsString();
		//std::cout<<FuncName<<"\n";
		//std::cout.flush();
		Stmt *FuncBody = f->getBody();
		//FuncBody->dump();
		//Stmt::child_range c = FuncBody->children();
		//Stmt::child_iterator *test = FuncBody->child_begin();
		//test->dump();
		//f->dump();
		//std::string bodystr = FuncBody.getAsString();
		//std::cout<<bodystr;
		//Rewrite.InsertText(FuncBody, "__SMC_Begin\n", true, true);
	//	SourceLocation ST = f->getSourceRange().getBegin();
	//	StmtIterator b = FuncBody->begin();
		//b++;
		//StmtIterator b = FuncBody;
		//while(*b!="{"){
		//	b++;	
		//}
		//CompoundStmt::body_iterator iter=FuncBody->_begin();
		for(Stmt::child_iterator b = FuncBody->child_begin(), e = FuncBody->child_end(); b!=e; ++b){
			
			//std::cout<<"hi\n";
			//std::cout<<b->getStmtClassName()<<"\n";
			if(b->getStmtClassName() == "DeclStmt"){
				//DeclStmt *d = (declStmt)*b;
				//for(Stmt::child_iterator d = b->child_begin(), f=b->child_end(); d!=f; ++d){
				
				//	std::cout<<"yes\n";
				//}
				DeclStmt *d = static_cast<DeclStmt*>(*b);
				if(d->isSingleDecl()){
					//std::cout<<"yess\n";
					Decl *d1 = d->getSingleDecl();
					//std::cout<<d1->getDeclKindName()<<"\n";
					NamedDecl *nd1 = static_cast<NamedDecl*>(d1);
					//std::cout<<"Name: "<<nd1->getNameAsString()<<"\n";
					//if(nd1->getNameAsString()=="bx"){
						VarDecl *vd1 = static_cast<VarDecl*>(nd1);
						Expr *init_expr = vd1->getInit();
						Expr *cast_ignored = init_expr->IgnoreImplicit();
						//std::cout<<"StmtClass: "<<cast_ignored->getStmtClassName()<<"\n";
						if(cast_ignored->getStmtClassName()=="PseudoObjectExpr"){
							PseudoObjectExpr *pseudo_object = static_cast<PseudoObjectExpr*>(cast_ignored);
							Expr *syntactic_form = pseudo_object->getSyntacticForm();
							//syntactic_form->dump();
							//std::cout<<"===============getResult=================================\n";
							Expr *result_expr = pseudo_object->getResultExpr();
							//result_expr->dump();
							//std::cout<<"~~~~~~~~~~~~~~~~~~~getResult~~~~~~~~~~~~~~~~~~~~~~~~~~~~\n";
							for(PseudoObjectExpr::semantics_iterator sb = pseudo_object->semantics_begin(), se = pseudo_object->semantics_end(); sb<se; ++sb){
								Expr *this_semantic = (*sb);
								//std::cout<<"=========================================\n";
								//this_semantic->dump();
								//std::cout<<"~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\n"<<this_semantic->getStmtClassName()<<"\n-------------------\n";
								if(this_semantic->getStmtClassName()=="OpaqueValueExpr"){
									OpaqueValueExpr *opaque_expr = static_cast<OpaqueValueExpr*>(this_semantic);
									//std::cout<<"--i\nopaque\n---\n";
									//opaque_expr->getSourceExpr()->dump();
									//std::cout<<"***********************************\n";
									Expr *opaque_dump = opaque_expr->getSourceExpr();
									if(opaque_dump->getStmtClassName()=="DeclRefExpr"){
										DeclRefExpr *blockidx_decl = static_cast<DeclRefExpr*>(opaque_dump);
										//std::cout<<"blockid name: "<<blockidx_decl->getNameInfo().getName().getAsString()<<"\n";
										if(blockidx_decl->getNameInfo().getName().getAsString() == "blockIdx"){
											if(result_expr->getStmtClassName()=="CallExpr"){
												CallExpr *callexpr_result = static_cast<CallExpr*>(result_expr);
												//std::cout<<"====================callee============================\n";
												//callexpr_result->getCallee()->dump();
												//std::cout<<"====================callee============================\n";
												Expr *callee_ignore_implicit = callexpr_result->getCallee()->IgnoreImplicit();
												//std::cout<<"callee member: "<<callee_ignore_implicit->getStmtClassName()<<"\n";
												if(callee_ignore_implicit->getStmtClassName() == "MemberExpr"){
													MemberExpr *callee_mem_expr = static_cast<MemberExpr*>(callee_ignore_implicit);
													//std::cout<<"Member Name: "<<callee_mem_expr->getMemberDecl()->getNameAsString()<<"\n";
													if(callee_mem_expr->getMemberDecl()->getNameAsString()=="__fetch_builtin_x"){
														const std::string block_x_string = "(int)fmodf((float)__SMC_chunkID, (float)__SMC_orgGridDim.x)";
														StringRef block_x_sref = StringRef(block_x_string);
														//std::cout<<"STRINGREF: "<<block_x_sref.str()<<"\n";
														Rewrite.ReplaceText(init_expr->getSourceRange(), block_x_sref);
													}
													else if(callee_mem_expr->getMemberDecl()->getNameAsString()=="__fetch_builtin_y"){
														const std::string block_y_string = "(int)(__SMC_chunkID/__SMC_orgGridDim.y)";
														StringRef block_y_sref = StringRef(block_y_string);
														Rewrite.ReplaceText(init_expr->getSourceRange(), block_y_sref);
													}



												}

											}
										}
									}	
								}
							}
						}
					//}
					//QualType qt = cast_ignored->getType();
					//std::cout<<"Qual type: "<<qt.getAsString()<<"\n";
					//cast_ignored->dump();
					//ImplicitCastExpr *imp_expr = static_cast<ImplicitCastExpr*>(init_expr);
					//imp_expr->dump();
					//std::cout<<"cast kind: "<<imp_expr->getCastKindName()<<"\n";
					//BinaryOperator *binary_init = static_cast<BinaryOperator*>(init_expr);
					/*if(binary_init->isAssignmentOp()){
						std::cout<<"assignment\n";
					}
					else{
						std::cout<<"false\n";
					}*/
					//b->dump();

				}
				

			}
		
		}
		if(!cudafunction){
			Rewrite.InsertText(FuncBody->getLocStart().getLocWithOffset(1), "\n__SMC_Begin\n", true, true);
			Rewrite.InsertText(FuncBody->getLocEnd(), "\n__SMC_End\n", true, true);
		//ArrayRef<ParmVarDecl*> p = f->parameters();
			Rewrite.InsertText(FuncBody->getLocStart().getLocWithOffset(-2)/*f->getLocEnd()*/, ", dim3 __SMC_orgGridDim, int __SMC_workersNeeded, int *__SMC_workerCount, int * __SMC_newChunkSeq, int * __SMC_seqEnds", true, true);
			cudafunction = true;
		}


	
      }
    }
  

private:
  Rewriter &Rewrite;
};


// Implementation of the ASTConsumer interface for reading an AST produced
// by the Clang parser. It registers a couple of matchers and runs them on
// the AST.
class MyASTConsumer : public ASTConsumer {
public:
  MyASTConsumer(Rewriter &R) : HandlerForFunction(R), HandlerForVarDecl(R), HandlerForCUDACall(R) {
    // Add a simple matcher for finding 'if' statements.
    Matcher.addMatcher(cudaKernelCallExpr().bind("cudaCall"), &HandlerForCUDACall);
    Matcher.addMatcher(functionDecl().bind("functionDecl"), &HandlerForFunction);
    Matcher.addMatcher(varDecl().bind("varDecl"), &HandlerForVarDecl);
    //Matcher.addMatcher(cudaKernelCallExpr().bind("cudaCall"), &HandlerForCUDACall);
    //Matcher.addMatcher(stmt().bind("statement"), &HandlerForHeaderFile);
    // Add a complex matcher for finding 'for' loops with an initializer set
    // to 0, < comparison in the codition and an increment. For example:
    //
    //  for (int i = 0; i < N; ++i)
  }

  void HandleTranslationUnit(ASTContext &Context) override {
    // Run the matchers when we have the whole TU parsed.
    Matcher.matchAST(Context);
  }

private:
  FunctionHandler HandlerForFunction;
  VarDeclHandler HandlerForVarDecl;
  CUDACallHandler HandlerForCUDACall;
  MatchFinder Matcher;
  //HeaderFileHandler HandlerForHeaderFile;
  //IncrementForLoopHandler HandlerForFor;
  
};

// For each source file provided to the tool, a new FrontendAction is created.
class MyFrontendAction : public ASTFrontendAction {
public:
  MyFrontendAction() {}
bool BeginSourceFileAction(CompilerInstance &CI) 
{
	      std::unique_ptr<IncludeHandler> HandlerForInclude(new IncludeHandler(TheRewriter));

              Preprocessor &pp = CI.getPreprocessor();
	      pp.addPPCallbacks(std::move(HandlerForInclude));
	      return true;
}
  void EndSourceFileAction() override {
    	
    //TheRewriter.getEditBuffer(TheRewriter.getSourceMgr().getMainFileID())
      //.write(llvm::outs());
    SourceManager &sm = TheRewriter.getSourceMgr();
    StringRef input = sm.getFileEntryForID(sm.getMainFileID())->getName();
    std::string file = input.str();
    std::string delimeter = ".";
    std::string name;
    std::string extension;
    size_t position = file.find(delimeter);
    name = file.substr(0, position);
    file.erase(0, position + delimeter.length());
    extension = file;
														
    std::string output = name+"_smc."+extension;
    std::error_code error_code;
    llvm::raw_fd_ostream outFile(output, error_code, llvm::sys::fs::F_None);
																										TheRewriter.getEditBuffer(TheRewriter.getSourceMgr().getMainFileID()).write(outFile);
																												outFile.close();
  }

  std::unique_ptr<ASTConsumer> CreateASTConsumer(CompilerInstance &CI,
						 StringRef file) override {
	
    TheRewriter.setSourceMgr(CI.getSourceManager(), CI.getLangOpts());
    return llvm::make_unique<MyASTConsumer>(TheRewriter);
  }

private:
  Rewriter TheRewriter;
};

int main(int argc, const char **argv) {
  CommonOptionsParser op(argc, argv, MatcherSampleCategory);
  ClangTool Tool(op.getCompilations(), op.getSourcePathList());

  return Tool.run(newFrontendActionFactory<MyFrontendAction>().get());
}
